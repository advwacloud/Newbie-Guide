# Notification API 開發入門

---

* 專案架構與多版本機制
* 核心程式邏輯
* 特殊形態-Webhook Group
* Shared Service和Service Broker簡介
  * 本文篇幅不夠只會簡單介紹概念, 細節敬請期待『Notification API - Shared Service』

---

## 專案架構與多版本機制

* API專案是基於loopback2.0去實作的
* loopback2.0原生沒支援同時多版本, 需要自己另外做
  * 多版本機制隔離了不同版本API的實作與介面, 確保代碼修改或phaseout時能維持正確性

### 專案目錄結構

![](/assets/4.PNG)

### Loopback remoteMethod

common下的model json定義的是同類別的api路徑  
![](/assets/9.PNG)

common下會有跟model json同名的js, 這支程式就是同一類api的主程式

* 例如 POST api/v1.5/Groups被調用時, 會進入Group.remoteMethod\('create'..., 
* 同一支檔案裡會有Group.create = async \(req, group\), 會實作api邏輯
* server下的model json定義的是api的請求與回應為何, 就像這裡的GroupCreateInstance\_\_v1\_5

![](/assets/10.PNG)

### 多版本API Doc

跑完npm run generate會建立所有版本的api doc, api doc需要加入版控  
![](/assets/6.PNG)

### 多版本與api doc相關工具

### ![](/assets/8.PNG)

多版本機制的進版與phaseout是透過複雜的流程完成, 所以請**務必使用多版本相關工具**來做api進版與淘汰

* createNewVerApi.js
  * createNewVerApi LATEST\_VERSION TARGET\_\_VERSION
  * createNewVerApi 1.5 2.0
* gen\_api\_doc.js
  * 不用帶參數, 跑完直接產出所有版本的SWAGGER API DOC
* phaseOutApi.js
  * 不用帶參數, 直接從最舊的版本PHASE OUT, 一次淘汰一板

## 核心程式邏輯

用建立一個排程的line group來說明發送訊息的代碼  \(共三步驟\)

### 步驟1. 建立Level

**API: api/v1.5/Levels**

**Request Body**

```
{
  "name": "每十分鐘發五則訊息，每則保存一天",
  "intervalTime": 10,
  "unit": "minute",
  "expTime": 1,
  "expUnit": "day",
  "batchCount": 5
}
```

建立level不看code, 因為就是很單純的建立config而已

### 步驟2. 建立Group

**API: api/v1.5/Groups**

**Request Body**

```
{  
   "name":"SayHelloToRoyLine",
   "description":"line test",
   "type":"line",
   "levelName":"每十分鐘發五則訊息，每則保存一天",
   "config":{  
      "template":"Hi {name}!"
   },
   "sendList":[  
      {  
         "firstName":"Roy",
         "lastName":"Chen",
         "target":"fso2ldoS0QfU38ZNSv0w1E3RQj4wCTu8MSeKWe5pNF8"
      }
   ]
}
```

**代碼說明**

檢查level是否存在, 若存在就用name去換level id

```
let results = await levelDao.getLevel({ name: group.levelName || '', instanceId });
let levelResult = results.rows[0];

// check level existed
if (!levelResult) {
  throw Error('no such level');
}
// assigned levelId to groupObj
group.levelId = levelResult.levelId;
```

將group細節寫入postgresql

```
 let groupDaoResp = await GroupDao.insertGroup(group, trans);
```

養鴿人定義鴿子, 並啟動鴿子

```
let defineOption = {
  id: groupDaoResp.groupId,
  type: groupDaoResp.type,
  interval: groupHelper.getTimeInSec(levelResult.intervalTime, levelResult.unit),
  batchCount: levelResult.batchCount,
  expireTime: groupHelper.getTimeInSec(levelResult.expTime, levelResult.expUnit)
};
await Const.pigeonBreeder.define(defineOption);
await Const.pigeonBreeder.start(defineOption.id);
```

這一步是很關鍵的一步, 抽象來說就是鴿子開始繞圈了, 具體來說則是, **Group每十分鐘會去檢查queue裡有沒有訊息, 有的話一次最多會拿五則訊息出來發送, 這個group的queue裡的訊息只能存活一天**

### 步驟3. 發送訊息 \(或將待送訊息放進Group Queue\)

**此api的每個object都會轉化為待送訊息放進Queue裡, 鴿子化的Group每十分鐘, 會去Queue裡抓5則訊息來發送**

**API: api/v1.5/Groups/send    **

**Request Body**

```
[
  {
    "groupId": "string",
    "subject": "string",
    "useTemplate": true,
    "variables": { "name": "Roy" },
    "sendList": [
      {
        "target": "string"
      }
    ]
  }
]
```

**代碼說明**

api會先預處理每一個object

* 防呆處理
* 用groupId去關聯拿到group的細節, 例如template是什麼

```
sendObjs = sendObjs.map(send => {
  let groupCfg = groupDaoResp.rows.find(
    group => group.groupId === send.groupId
  );
  if (!send.sendList) {
    delete send.sendList;
  }
  return Object.assign({}, _.cloneDeep(groupCfg), send);
})

...
...
return genSendingObj(sendObj);
```

genSendingObj function裡, 會去處理每一種type預處理的邏輯, line的話很單純, 就是在message加上固定的postfix字樣

    case Const.Group.Type.Line:
      options = {
        sendList: targetList,
        useTemplate: useTemplate,
        variables: useTemplate ? variables : null
      };
      if (useTemplate && config.template) {
        options.template = `${config.template}\n\n${Const.NotifyNote}`;
      } else if (message) {
        options.message = `${message}\n\n${Const.NotifyNote}`;
      }
      result = await sendWrapperFunc(options, type, groupId, targetList);
      return result;

在genSendingObj function裡處理完每一種type各自的預處理邏輯後, 會將參數傳入共用的sendWrapperFunc function來做發送

* 如果有帶groupId的, 就調用pigeonBreeder.send\(groupId, options\)
  * api/v1.5/Groups/send 會用到
* 如果沒帶groupId的, 就調用pigeonBreeder.try\(options\)
  * api/v1.5/Groups/test 會用到
  * api/v1.5/Groups/directsend 會用到
* send/test/directsend 三支api差別
  * send可以排程, 有鴿子機制
  * test/directsend都是立即發送, 無鴿子機制
  * test可以測試template功能, directsend沒有template功能 

```
function sendWrapperFunc (options, type, groupId, targetList) {
  if (groupId) {
    // for API: POST /Groups/send
    return Const.pigeonBreeder
      .send(groupId, options)
      .then(res => {
        let processedResp = sendFullfillFunc(res, type, groupId);
        return processedResp;
      })
      .catch(err => {
        let processedErr = sendRejectFunc(err, targetList, groupId, type);
        return processedErr;
      });
  } else {
    // for API: POST /Groups/test
    options.type = type;

    return Const.pigeonBreeder
      .try(options)
      .then(res => {
        let processedResp = sendFullfillFunc(res, type, groupId);
        return processedResp;
      })
      .catch(err => {
        let processedErr = sendRejectFunc(err, targetList, groupId, type);
        return processedErr;
      });
  }
}
```

## 特殊形態-Webhook Group

* webhook Group不是發送訊息, 而是觸發預先設定的http api
* 例如可以預先設定webhook為line的發訊息api, 
* 也就是說, 有了webhook group這功能, 可以取代其他訊息服務的group, 
* 但比較不方便就是了, 因為其他訊息服務的group我們已經把複雜的部份封裝起來, 只需要填必要資訊就好
* 即便webhook用途跟其他group不同, 但以操作流程跟排程機制來說, 還是跟其他group相同的
  * 所以代碼的部分還是跟上述核心核心程式邏輯共用

先看下建立webhook group的request body

```
{
  "name": "webhookToLine",
  "description": "line msg",
  "type": "webhook",
  "levelName": "Critical",
  "sendList": [
    {
      "target": "send line message",
      "method": "post",
      "url": "https://notify-api.line.me/api/notify",
      "timeout": 10,
      "headers": {
          "Content-Type": "X-www-form-urlencoded",
          "Authorization": "Bearer fso2ldoS0QfU38ZNSv0w1E3RQj4wCTu8MSeKWe5pNF"
      },
      "contentType": 2,
      "body": {
        "message": "Hi {name}"
      }
    }
  ]
}
```

接這來看下api/v1.5/Groups/send, 跟line group一樣會跑到genSendingObj function裡, 但webhook預處理有一點注意

* useTemplate一定是true

```
    case Const.Group.Type.Webhook:
      options = {
        sendList,
        useTemplate: true,
        variables: variables || {}
      };
      result = await sendWrapperFunc(options, type, groupId, targetList);
      return result;
```

所以看懂line的核心程式邏輯, webhook的程式流程也是相通的

## Shared Service和Service Broker簡介

### 什麼是Shared Service

* Shared service就是, 所有notification使用者都共用同一個notification server, disk/ram/流量/db等等是共享的, 租戶不需要自己佈署notification
* 而我們現notification現行發佈的方式為dedicated service, 或稱standalone app, 就是要用的話要在自己的namespace佈署notification, 流量或資源不共享

### 什麼是Service Broker

* 平台如cf或k8s都會透過我們自己維護的service broker來建立每個客戶自己的service instance
  * cf/k8s都會根據service broker spec去實作相關接口, 但實作細節可能會有差異
  * [https://github.com/openservicebrokerapi/servicebroker](https://github.com/openservicebrokerapi/servicebroker)
* service instance間資訊是隔離的
* 也就是說, a instance看不到b instance的group list, 但這兩個instance實際是共用同一pg database的

### Notiication Shared Service整合近況

* notification shared service架構 \(多租戶架構\)已實作完成 \(包含db schema\)
* notification service broker實作完成 \(koa server\)
* CF shared service測試完成
* k8s尚未測試, 因為要跟平台確認相關細節, 包含佈署方式/broker spec版本/特殊規範



