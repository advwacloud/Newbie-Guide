# 開發環境 - notification k8s部署範例

## Intro

* 介紹如何使用helm chart來部署datahub到k8s
  * chart: 包含了k8s各個資源的template\(deployment template/service template/ingress template...\), 部署時會配合values.yaml將值注入template \(values.yaml也會有template\)
  * helm: 部署/更新/查詢 chart
  * advantage: 參數化部屬/一次部署全部相關資源/一次刪除全部相關資源
* 注意: 本篇chart username/password我都挖掉了, 要自己填上去

## Prerequisite

* helm v3.x
* kubectl v1.x
* docker
* harbor
  * [https://harbor.arfa.wise-paas.com/harbor](https://harbor.arfa.wise-paas.com/harbor)

## Flow \(6 Steps\)

1. 修改代碼, 重做image並推上harbor
2. 使用本機helm註冊harbor上的chart repo \(只有第一次需要\)
3. 如果chart也有修改, 也需要把它推上harbor
4. 去harbor上拿到預設的 chart values.yaml內容, 依據不同環境修改內容
5. 搭配values.yaml, 使用helm install將app部署上k8s
6. 下指令檢查部署狀態

---

## Step 1 - 每次改code\(或commit新東西\)都要重新做image上harbor, 我都在linux做 \(因為個人電腦防毒不能裝docker\)

\[linux\] /home/wadev3/roy/buildspace/notification

改完code \(或pull完\), 直接在專案根目錄跑指令, **指令可以一鍵做image+上傳到harbor**:

```
flag="harbor.arfa.wise-paas.com/notification-dev/notification-portal:1.0.26-dev" \
&& sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
&& sudo docker push $flag
```

## Step 2 - 在本機helm註冊chart repo, 此步驟跑過一次就好

helm repo add --username harborUser --password harborPwd notification-dev [https://harbor.arfa.wise-paas.com/chartrepo/notification-dev](https://harbor.arfa.wise-paas.com/chartrepo/notification-dev)

* notification-dev是helm在本機註冊的別名
* helm repo ls

## Step 3 - 如果chart也有修改, 也需要把它推上harbor \(若無修改請略過\)

helm push notification-chart/ notification-dev

## Step 4 - 準備helm install file

**標準流程: **

* **去harbor找到要用的chart, 複製values內容, 修改值\(例如env.\*\)**
* 需要修改的地方有

  * portal/worker的image細節 \(repo/tag\)
  * global.database.secretName
  * * 多合一的secret
  * global.url.host

    * 規則是 .$namespace-name.$clustername.internal

  * ingress.hosts.host

    * external url裡可讓開發者修改的部分, 可包含dash, 預設是portal-notification, 如果space已經有, 請改別的名子, 避免url衝突, 例如portal-notification-dev

** url 內外部映射範例 **

**example 1**

* url.host: .scada.slave04.internal
* ingress.hosts.host: portal-notification
* external: [https://portal-notification-scada-slave04.es.wise-paas.cn](https://portal-notification-scada-slave04.es.wise-paas.cn)

**example 2**

* url.host: .scada.slave04.internal
* ingress.hosts.host: portal-notification-roy
* external: [https://portal-notification-roy-scada-slave04.es.wise-paas.cn](https://portal-notification-roy-scada-slave04.es.wise-paas.cn)

![](/assets/03.png)

## Step 5 - 佈到k8s上 helm install

helm install CHART\_NAME\_ON\_SERVER notification-dev/notification --version 1.0.26 -f values.yaml

> helm install notificationroy notification-dev/notification --version 1.0.26 -f slave04-scada-notification-install.yaml

CHART\_NAME\_ON\_SERVER請佈notification以外的名子 \(共用的notification我取這個名子\)

## Step 6 - 下一些指令檢查狀態

```
## 列出所有佈到k8s上的helm chart
helm ls

## 列出某一個install完的chart狀態
helm CHART_NAME status

## 列出所有pod (可透過-n指定namespace, 或是直接在k8s config指定namespace)
kubectl get pod

## 列出pod的log
kubectl log POD_NAME
```



