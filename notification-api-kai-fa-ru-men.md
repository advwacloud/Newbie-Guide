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

發送訊息

* Group.send

* Group.test

* Group.directsend

* genSendingObj

* Const.pigeonBreeder

  * .send

  * .try



