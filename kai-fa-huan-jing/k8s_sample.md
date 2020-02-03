# 開發環境 - k8s部署範例

注意: 本篇chart username/password我都挖掉了, 要自己填上去


## Part 1 - 如果每次改code(或commit新東西)都要重新做image上harbor, 我都在.57做 (因為個人電腦防毒不能裝docker)

[.57] /home/wadev3/roy/buildspace/Front-End
[.57] /home/wadev3/roy/buildspace/scada-dataworker

改完code (或pull完), 直接在專案根目錄跑指令, **指令可以一鍵做image+上傳到harbor**:

**portal
**flag="harbor.arfa.wise-paas.com/scada-dev/portal-scada:1.3.39-roydev" \
&& sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
&& sudo docker push $flag

**worker
**flag="harbor.arfa.wise-paas.com/scada-dev/scada-dataworker:1.3.31-roydev" \
&& sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
&& sudo docker push $flag


## Part 2 - 在本機helm註冊chart repo, 此步驟跑過一次就好

helm repo add --username harborUser --password harborPwd scada-dev https://harbor.arfa.wise-paas.com/chartrepo/scada-dev

## Part 3 - 準備helm install file

**標準流程: 去harbor找到要用的chart, 複製values內容, 修改值(ex. datacenter/cluster....)**

但這裡直接提供範本, 只需要改image tag就好

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
      cpu: 300m
      memory: 512Mi
      ephemeral-storage: 200M
    requests:
      cpu: 200m
      memory: 256Mi
      ephemeral-storage: 200M
  image:
    repository: harbor.arfa.wise-paas.com/scada-dev/portal-scada
    tag: 1.3.39-roydev
    pullPolicy: Always

worker:
  name: worker
  replicaCount: 1
  resources:
    limits:
      cpu: 300m
      memory: 256Mi
      ephemeral-storage: 200M
    requests:
      cpu: 200m
      memory: 256Mi
      ephemeral-storage: 200M
  image:
    repository: harbor.arfa.wise-paas.com/scada-dev/scada-dataworker
    tag: 1.3.31-roydev
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
  datacenter: "local"
  cluster: "slave04"
  workspace_id: "20a957f4-0bf9-4faf-90cd-694919cd4b68"
  namespace: "scada"
  sso_url: "http://api.sso.master.internal"
  sso_external_url: ""
  dccs_url: "https://api-dccsv4-master.es.wise-paas.cn"
  pn: "9806WAC010"
  pn_quantity: 1000
  lic_url: "https://api-license-master.es.wise-paas.cn"
  lic_interval: 3600000
  lic_key: ""
  notification_url: ""
  tech_url: ""
  mp_url: ""
  dashboard_url: ""
  rabbitmq_service_instance_name: ""
  amqp_prefetch: 10
  MESSAGE_RULE: ""
  default_domain: ""
  dccs_service_name: ""

database:
  secretName: scada-allsvc-secret

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

url:
  host: .scada.slave04.internal

ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: portal-scada
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

