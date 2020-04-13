# Kafka deploy

## Deploy

建一個namespace, 名子取kafka, namespace不要設quota, quota繼承workspace的
![](/assets/01.png)

本地k8s config的namespace指到kafka

去harbor抓kafka chart, 然後解壓縮
https://harbor.arfa.wise-paas.com/harbor/projects/24/helm-charts/kafka/versions/1.1.5

編輯deploy.sh, 下圖圈紅線的地方是已經修改過的
![](/assets/02.png)

在chart根目錄執行


```
$ ./deploy.sh kafkatest
```

看到下面這個hint, 代表佈成功


![](/assets/0410_03.png)

接著一直反覆執行 `kubectl get all`, statefulset都佈完成代表所有資源已成功運行

![](/assets/0410_04.png)


## 驗證

登入management portal建個topic

平台開的測試環境IP是172.21.92.173, 下kubectl get all找到kafkatest-kafka-manager (NodePort)的port為32556, 接著在瀏覽器訪問172.21.92.173:32556

密碼的話請下指令: helm status kafkatest, 再用提示的指令拿到帳密

```
Kafka Web Manager authentication:
username:
kubectl get secret kafkatest-kafka-manager -n kafka -o jsonpath='{.data.basicAuthUsername}' | base64 -d
password:
kubectl get secret kafkatest-kafka-manager -n kafka -o jsonpath='{.data.basicAuthPassword}' | base64 -d

```

登入portal後, 創個topic
![](/assets/0410_05.png)

接著開啟Conductor

要先組一個url, 規則為`$KFK_BROKER_POD_NAME.$KFK_BROKER_HEADLESS_SVC_NAME.$NAMESPACE.svc.cluster.local`

以這篇範例來說, 就是

kafkatest-0.kafkatest-kafka-headless.kafka.svc.cluster.local

接著去你電腦的hosts file加上一行

`172.21.92.173 kafkatest-0.kafkatest-kafka-headless.kafka.svc.cluster.local`

![](/assets/0410_06.png)


接著在Conductor設定zk和kfk broker, external ip請下kubectl get all去找, Additional Properties用下面這段

```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="admin" password="admin-secret";
serviceName="Kafka"
```

![](/assets/0410_07.png)

接著就可以測試producer/consumer了

![](/assets/0410_08.png)

## Prometheus Test

要測Prometheus要先請平台的人在站點安裝Prometheus

Prometheus會自動找尋服務, 所以不須其他設定, 就可以看到像下面這張圖
![](/assets/041301.PNG)

接著可以下查詢語法, 取出各項量測指標, 這裡參考Kenny投影片, 查詢特定topic的每秒訊息量統計
`sum(kafka_server_brokertopicmetrics_messagesinpersec_count{job="kafkatest"}) by (topic)`

![](/assets/041302.PNG)

![](/assets/041303.PNG)





