## 各類k8s config sample

### notification helm install values.yaml
```
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
notification:
  name: notification
  containerPort: 3000
  replicaCount: 1
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
      ephemeral-storage: 200M
    requests:
      cpu: 100m
      memory: 256Mi
      ephemeral-storage: 200M
  image:
    repository: harbor.arfa.wise-paas.com/notification-dev/portal-notification
    tag: 1.0.26-dev
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
  pn: "9806WPSC02"
  pn_quantity: 1
  lic_url: "https://api-license-master.es.wise-paas.cn"
  lic_interval: 3600000
  lic_key: ""
  tech_url: ""
  amqp_prefetch: 10
  default_domain: ""
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
    - host: portal-notification
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