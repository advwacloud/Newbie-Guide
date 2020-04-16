# Kafka deploy

## Deploy

建一個namespace, 名子取kafka, namespace不要設quota, quota繼承workspace的
![](/assets/01.png)

本地k8s config的namespace指到kafka

去harbor抓kafka chart, 然後解壓縮 (`tar xvf FileName.tar`)

https://harbor.arfa.wise-paas.com/harbor/projects/68/helm-charts/kafka/versions/1.1.5

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

## 使用Kafka Broker Container內建工具

這裡是補充Kenny投影片裡面`/usr/bin/kafka-console-producer` 跟 `/usr/bin/kafka-console-consumer` 操作細節

### 進去container

```
kubectl exec -it kafkatest-0 --container kafka-broker -- /bin/bash
```

### 把container的kafka-run-class複製到本地

```
kubectl cp kafkatest-0:usr/bin/kafka-run-class kafka-run-class --container kafka-broker
```

### 把本地的修改完的kafka-run-class複製回container

把kafka-run-class下面這三行註解掉
```
# JMX port to use
#if [  $JMX_PORT ]; then
#  KAFKA_JMX_OPTS="$KAFKA_JMX_OPTS -Dcom.sun.management.jmxremote.port=$JMX_PORT "
#fi
```

```
kubectl cp kafka-run-class kafkatest-0:usr/bin/kafka-run-class --container kafka-broker

## in container 
chmod +x /usr/bin/kafka-run-class
```

### 把producer.properties複製進container

```
kubectl cp producer.properties kafkatest-0:etc/kafka/producer.properties --container kafka-broker
```

### 把consumer.properties複製進container

```
kubectl cp consumer.properties kafkatest-0:etc/kafka/consumer.properties --container kafka-broker
```

### 把kafka_client_jaas.conf複製進container

```
kubectl cp kafka_client_jaas.conf kafkatest-0:etc/kafka/kafka_client_jaas.conf --container kafka-broker
```

### producer發送

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf" \
&& ./usr/bin/kafka-console-producer --broker-list 10.233.23.176:9092 --topic roy02 --producer.config /etc/kafka/producer.properties
```

### consumer接收

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf" \
&& ./usr/bin/kafka-console-consumer --bootstrap-server 10.233.11.67:9092 --topic roy1 --from-beginning --consumer.config /etc/kafka/consumer.properties
```

### 補充 - 壓力測試工具

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf" \
&& ./usr/bin/kafka-producer-perf-test --num-records 10000000 --topic roy1 --throughput 100000 --record-size 100 --producer-props bootstrap.servers=10.233.11.67:9092 --producer.config /etc/kafka/producer.properties 
```






