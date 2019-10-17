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




