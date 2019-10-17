# 開發環境 - kubernetes

---

* docker image hub & chart repo

  * [https://harbor.wise-paas.io](https://harbor.wise-paas.io/)

### Kubectl
- 下載kubectl.exe, [從這裡載](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows), 放置於任意位置, ex. C:\Users\boshen.chen\Documents\k8s

![](/assets/kubectlpath.png)
- 在環境變數Path加入kubectl所在目錄
![](/assets/kubectlpathenv_mask.png)
- 使用cluster config, 這會是k8s team給我們的, 讓kubectl可以跟遠端環境互動, 該config會長的像這樣

```
apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  name: development
- cluster:
  name: scratch

users:
- name: developer
- name: experimenter

contexts:
- context:
  name: dev-frontend
- context:
  name: dev-storage
- context:
  name: exp-scratch
```

- 可以在current-context先設定好要用哪個namespace, 否則會使用default這個namespace

```
contexts:
- context:
    cluster: test
    user: user
    namespace: "scada-dev"
  name: test
current-context: test
```
- 最後則是在環境變數KUBECONFIG設定該config的位置, 要注意的是, 除了路徑, 要指定到檔名

![](/assets/kubeconfig.PNG)


- 都設定好後就可以使用kubectl來跟cluster互動了
![](/assets/kubectlusage.PNG)

### Helm
* 安裝Chocolatey, 這是一款Windows的套件管理工具,安裝方式請參考這裡 (https://chocolatey.org/install#installing-chocolatey)
* 安裝好Chocolatey之後, 執行這行指令, 即可安裝helm
```
choco install kubernetes-helm
```
* 安裝好後, 可以用helm ls簡單測試一下
![](/assets/helmls.PNG)
* 新增harbor chart repo
```
helm repo add --username {USERNAME} --password {PASSWORD} scada https://harbor.wise-paas.io/chartrepo/scada
```
* 透過chart部署scada portal/worker到k8s cluster上 (以下指令是Win cmd用的)
```
(
helm install scada/scada --name scada --username {USERNAME} --password {PASSWORD} --version 1.1.31 ^
--set ingress.hosts={portal-scada-develop.ensaas190920104500.wise-paas.com.cn} ^
--set portal.resources.requests.cpu=100m ^
--set worker.resources.requests.cpu=100m ^
--set imageCredentials.username={USERNAME} ^
--set imageCredentials.password={PASSWORD} ^
--set envs.org_name=EnSaaS-190826113 ^
--set envs.org_id=a7928e16-cfdc-4e92-8130-7b316851af30 ^
--set envs.space_name=scada ^
--set envs.space_id=75a5b889-c89d-11e9-bcc4-52df743a3a14 ^
--set envs.application_id=apppppp-pppp-pppp3-44444 ^
--set envs.sso_url="https://portal-sso.wise-paas.com.cn" ^
--set envs.dccs_url="https://api-dccs-190826113000.wise-paas.com.cn"
)
```
* 部署成功後, 可以透過 `helm status scada` 來查看結果
![](/assets/helmstatus.PNG)

