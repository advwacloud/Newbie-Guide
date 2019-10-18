# 開發環境 - kubernetes

---

* docker image hub & chart repo

  * [https://harbor.wise-paas.io](https://harbor.wise-paas.io/)

### Kubectl

* 下載kubectl.exe, [從這裡載](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows), 放置於任意位置, ex. C:\Users\boshen.chen\Documents\k8s

![](/assets/kubectlpath.png)

* 在環境變數Path加入kubectl所在目錄
  ![](/assets/kubectlpathenv_mask.png)
* 接下來要設定cluster config, 這會是k8s team給我們的, 讓kubect/helml可以跟遠端環境互動, 該config會長的像這樣

```
apiVersion: v1
kind: Config
...
...
```

* 可以在current-context先設定好要用哪個namespace, 否則會使用default這個namespace, 每次指令都要指定namespace很麻煩

```
contexts:
- context:
    cluster: test
    user: user
    namespace: scada-dev
  name: test
current-context: test
```

* 最後則是在環境變數KUBECONFIG設定該config的位置, 要注意的是, 除了路徑, 要指定到檔名

![](/assets/kubeconfig.PNG)

* `kubectl config view` 可以檢查是否有設定正確
* 都設定好後就可以使用kubectl來跟cluster互動了
  ![](/assets/kubectlusage.PNG)

### Helm

* 安裝Chocolatey, 這是一款Windows的套件管理工具,[安裝方式請參考這裡](https://chocolatey.org/docs/installation#install-with-cmdexe)
  * 我是用cmd去裝，要使用管理者權限去開cmd
* 安裝好Chocolatey之後, 執行這行指令, 即可安裝helm \(也需要管理者權限\)
  ```
  choco install kubernetes-helm
  ```
* 安裝好後, 可以用helm ls簡單測試一下, 也可以用`helm status {chart name}` 來測試
  ![](/assets/helmls.PNG)
* `helm init` 進行helm client/server初始化
* 新增harbor chart repo
  ```
  helm repo add --username {USERNAME} --password {PASSWORD} scada https://harbor.wise-paas.io/chartrepo/scada
  ```
* 透過chart部署scada portal/worker到k8s cluster上 \(以下指令是Win cmd用的\)
  ```
  (
  helm install scada/scada --name scada --username {USERNAME} --password {PASSWORD} --version 1.1.33 ^
  --set imageCredentials.username={USERNAME} ^
  --set imageCredentials.password={PASSWORD} ^
  --set portal.resources.requests.cpu=100m ^
  --set worker.resources.requests.cpu=100m ^
  --set ingress.hosts={portal-scada-develop.ensaas190920104500.wise-paas.com.cn} ^
  --set envs.org_name=EnSaaS-190903164 ^
  --set envs.org_id=53a7c740-a5c4-4d9e-ac81-38537baf745b ^
  --set envs.space_name=scada ^
  --set envs.space_id=9a965c63-507f-49fc-8f82-7e63e22e282c ^
  --set envs.application_id=apppppp-pppp-pppp3-44444 ^
  --set envs.sso_url="https://portal-sso-ensaas-operation-hk.jx.wise-paas.com.cn" ^
  --set envs.dccs_url="https://api-dccs-ensaas-operation-hk.jx.wise-paas.com.cn"
  )
  ```
* 部署成功後, 可以透過 `helm status scada` 來查看結果
  ![](/assets/helmstatus.PNG)

### 手動build測試用的image到habor

* **勿將**手動build的image放倒scada repo, 而是要放到scada-dev repo
* 172.22.2.57為我們的build machine, 大家可以上去產生自己測試用的docker image
* clone專案後, 在專案根目錄執行以下命令即可產生docker image, 並上傳至harbor
* scada portal test image
  ```
  flag="harbor.wise-paas.io/scada-dev/portal-scada:1.3.33-dev" \
  && sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
  && sudo docker push $flag
  ```
* scada worker test image
  ```
  flag="harbor.wise-paas.io/scada-dev/scada-dataworker:1.3.26-dev" \
  && sudo docker build -t=$flag --no-cache=true -f="./dockerfiles/localbuild.dockerfile" . \
  && sudo docker push $flag
  ```

### 置換scada chart預設的image

* 以scada chart@1.1.33為例, portal image版本預設為1.3.33
* 開發期間如果要改成其他image來測試, 只要換掉預設的image的repo和version即可
  ```
  --set portal.image.repository=harbor.wise-paas.io/scada-dev/portal-scada ^
  --set portal.image.tag=1.3.33-dev ^
  ```

### K8S Services And Secret
broker/class/plan是平台的人需要幫我們建好, 我們能操作的是建立service instance和servcie binding

`kubectl get clusterservicebroker`
![](/assets/svnbrokerresult.PNG)
`kubectl get clusterserviceclass`
![](/assets/svcclasresult.PNG)
`kubectl get clusterserviceplan`
![](/assets/svcplanresult.PNG)

假設我們要建立建立mongo的service instance和binding, 可以準備yaml檔, 配合kubectl去執行

create-mongo-instance.yaml
```
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ServiceInstance
metadata:
  name: mongo-instance
  namespace: scada-dev
spec:
  clusterServiceClassExternalName: mongodb-shared-broker
  clusterServicePlanExternalName: Shared
```
`kubectl create -f create-mongo-instance.yaml`

create-mongo-binding.yaml
```
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ServiceBinding
metadata:
  name: mongo-binding
  namespace: scada-dev
spec:
  instanceRef:
    name: mongo-instance
  secretName: mongo-credential
```
`kubectl create -f create-mongo-binding.yaml`

建立好mongo binding之後, secret也會一併建立好

`kubectl get secret`
![](/assets/k8ssecretresult.PNG)

也可以透過kubectl直接看secert內容, secret都是以base64編碼
`kubectl get secret mongo-credential -o yaml`
![](/assets/secretmongo.PNG)

最後要介紹一下, k8s是如何將這些secret以環境變數的方式注入給app

chart deployment裡有一段是橋接secret和app環境變數

```
- name: MONGO_URI
  valueFrom:
    secretKeyRef:
      name: mongo-credential
      key: uri
      optional: true
```

在scada portal裡就會以 `uri: process.env.MONGO_URI` 這種方式去接secret的uri