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

密碼的話請下指令: helm status kafka

```
Kafka Web Manager authentication:
username:
kubectl get secret kafkatest-kafka-manager -n kafka -o jsonpath='{.data.basicAuthUsername}' | base64 -d
password:
kubectl get secret kafkatest-kafka-manager -n kafka -o jsonpath='{.data.basicAuthPassword}' | base64 -d

```