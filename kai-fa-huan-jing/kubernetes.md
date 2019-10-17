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

