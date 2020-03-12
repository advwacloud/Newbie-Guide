# Notification API 開發入門

---

![](/assets/1.PNG)![](/assets/2.PNG)

## 程式邏輯

### 建立Level

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

### 建立Group

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

發送訊息

* Group.send

* Group.test

* Group.directsend

* genSendingObj

* Const.pigeonBreeder

  * .send

  * .try



