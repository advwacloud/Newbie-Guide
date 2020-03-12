# Notification API 開發入門

---

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

#### api/v1.5/Groups/send

#### 此api的每個object都會轉化為待送訊息放進Queue裡, Group每十分鐘, 會去Queue裡抓5則訊息來發送

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



