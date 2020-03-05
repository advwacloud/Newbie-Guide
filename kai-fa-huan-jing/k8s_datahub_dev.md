# 開發環境 - datahub k8s部署範例

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

\[linux\] /home/wadev3/roy/buildspace/datahub-portal

\[linux\] /home/wadev3/roy/buildspace/datahub-dataworker

改完code \(或pull完\), 直接在專案根目錄跑指令, **指令可以一鍵做image+上傳到harbor**:

**portal**

```
flag="harbor.arfa.wise-paas.com/datahub-dev/datahub-portal:2.0.1" \
&& sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
&& sudo docker push $flag
```

**worker**

```
flag="harbor.arfa.wise-paas.com/datahub-dev/datahub-dataworker:2.0.1" \
&& sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
&& sudo docker push $flag
```

## Step 2 - 在本機helm註冊chart repo, 此步驟跑過一次就好

```
helm repo add --username harborUser --password harborPwd datahub-dev https://harbor.arfa.wise-paas.com/chartrepo/datahub-dev
```

* datahub-dev是helm在本機註冊的別名
* helm repo ls

## Step 3 - 如果chart也有修改, 也需要把它推上harbor

helm push datahub-chart/ datahub-dev

## Step 4 - 準備helm install file

**標準流程: **

* **去harbor找到要用的chart, 複製values內容, 修改值\(例如env.\*\)**
* 需要修改的地方有

  * portal/worker的image細節 \(repo/tag\)
  * global.database.secretName
  * * 多合一的secret
  * url.host

    * 規則是 .$namespace-name.$clustername.internal

  * ingress.hosts.host

    * external url裡可讓開發者修改的部分, 可包含dash, 預設是portal-datahub, 如果space已經有, 請改別的名子, 避免url衝突, 例如portal-datahub-dev

** url 內外部映射範例 **

**example 1**

* url.host: .scada.slave04.internal
* ingress.hosts.host: portal-datahub
* external: [https://portal-datahub-scada-slave04.es.wise-paas.cn](https://portal-scada-scada-slave04.es.wise-paas.cn)

**example 2**

* url.host: .scada.slave04.internal
* ingress.hosts.host: portal-datahub-roy
* external: [https://portal-datahub-roy-scada-slave04.es.wise-paas.cn](https://portal-scada-roy-scada-slave04.es.wise-paas.cn)

這份文件裡直接提供範本,

檔名我存成slave04-scada-scada-install.yaml

```
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

portal:
  name: portal
  containerPort: 3000
  replicaCount: 1
  resources:
    limits:
      cpu: 100m
      memory: 512Mi
      ephemeral-storage: 500Mi
    requests:
      cpu: 100m
      memory: 256Mi
      ephemeral-storage: 500Mi
  image:
    repository: harbor.arfa.wise-paas.com/datahub-dev/datahub-portal
    tag: 2.0.0
    pullPolicy: Always

worker:
  name: worker
  replicaCount: 1
  resources:
    limits:
      cpu: 100m
      memory: 256Mi
      ephemeral-storage: 500Mi
    requests:
      cpu: 100m
      memory: 256Mi
      ephemeral-storage: 500Mi
  image:
    repository: harbor.arfa.wise-paas.com/datahub/datahub-dataworker
    tag: 2.0.0
    pullPolicy: Always

imageCredentials:
  registry: harbor.arfa.wise-paas.com
  username: ""
  password: ""

nameOverride: ""
fullnameOverride: ""

command:
args:

# envs is a map to load env
envs:
  NODE_ENV: production
  https_isset: "true"
  pn: "9806WAC010"
  pn_quantity: 1
  lic_interval: 3600000
  lic_key: ""
  notification_url: ""
  tech_url: ""
  dashboard_url: ""
  amqp_prefetch: 10
  MESSAGE_RULE: ""


global:
  database:
    secretName: datahub-all-in-one-secret
  url:
    host: .scada.slave04.internal
  ensaasApps:
    apiSso:
      internalUrl: ""
      externalUrl: ""
    apiMg:
      internalUrl: ""
      externalUrl: ""
    apiDccs:
      internalUrl: ""
      externalUrl: ""
    apiLicense:
      internalUrl: ""
      externalUrl: ""
    apiListingsystem:
      internalUrl: ""
      externalUrl: "http://api-listingsystem-master.es.wise-paas.cn/v1/datacenter"
    apiService:
      internalUrl: ""
      externalUrl: ""
    ensaas:
      datacenterCode: ""
      internalUrl: ""
      externalUrl: ""

#livenss and readness
livenessProbe:
  enabled: false
  initialDelaySeconds: 30
  periodSeconds: 10
  httpGet:
    path : /

readinessProbe:
  enabled: false
  initialDelaySeconds: 5
  periodSeconds: 10
  httpGet:
    path : /


ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: portal-datahub
      paths: ['/']

  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

service:
  type: ClusterIP
  #targetPort is the pod port
  targetPort: 3000
  #port is the service port
  port: 80

nodeSelector: {}

tolerations: []

affinity: {}
```

## Step 5 - 佈到k8s上 helm install

helm install CHART\_NAME\_ON\_SERVER datahub-dev/datahub --version 2.0.0 -f values.yaml

> helm install datahub-roy datahub-dev/datahub --version 2.0.0 -f slave04-scada-scada-install.yaml

CHART\_NAME\_ON\_SERVER請佈datahub以外的名子 \(共用的datahub我取這個名子\)

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



