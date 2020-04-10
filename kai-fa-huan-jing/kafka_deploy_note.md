# Kafka deploy

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

d

