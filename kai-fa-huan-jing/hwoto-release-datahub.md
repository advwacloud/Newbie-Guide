# Release DataHub教學

---

1. 準備worker/portal release note草稿給和益
2. 將datahub-version的大小版號更新
3. portal/worker分別打TAG, 觸發CI自動建立/推送image到林口harbor \(datahub project\)
4. 在chart專案也打TAG\(同大版號\), 觸發CI自動建立/推送chart到林口harbor \(datahub project\)
5. image/chart都建好後, 通知和益
6. 和益會發正式Release Note, 並向平台\(窗口孫迪\)提交Chart

---

## Step1. 準備worker/portal release note草稿給和益

* 慣例是會拿專案裡的CHANGELOG.md稍作整理後傳給和益做最後修改
  * 例如一些內部的修改, 會是備註類文字的可以拿掉

## Step2. 將datahub-version的大小版號更新

* 目前會請和益更新

* 這裡指的大小版號為

  * 大版號: DataHub版號

  * 小版號: portal/worker各自專案的版號

* [http://advgitlab.eastasia.cloudapp.azure.com/DataHub/datahub-version/blob/master/version.json](http://advgitlab.eastasia.cloudapp.azure.com/DataHub/datahub-version/blob/master/version.json)

* 內容長的像這樣

  * ```
     ...
     "2.0.1": {
        "datahub-dataworker": "2.0.1",
        "datahub-portal": "2.0.1",
        "release-date": "2020-03-04"
      },
      ...
    ```

## Step3. portal/worker分別打TAG, 觸發CI自動建立/推送image到林口harbor \(datahub project\)

* portal/worker release時, 都會merge回master branch, 並打上Tag, 也就是上述的小版號
* 我們有串jenkins CI PIpeline, 會自動build image, 並傳到林口harbor下的datahub project

## Step4. 在chart專案也打TAG\(同大版號\), 觸發CI自動建立/推送chart到林口harbor \(datahub project\)

* [http://advgitlab.eastasia.cloudapp.azure.com/DataHub/pipeline\_datahub\_helmchart\_build](http://advgitlab.eastasia.cloudapp.azure.com/DataHub/pipeline_datahub_helmchart_build)

* 每個datahub release會搭配一個chart

* 所以要clone helm chart專案, 並打上tag, 版號為大版號

* helm chart不一定每次release都會修改, 但還是要build一個對應版號的chart

  * 也就是說, 同一個commit可能會打多個Tag

## Step5. image/chart都建好後, 通知和益

跟和益說image/chart都建好了

## Step6. 和益會發正式Release Note, 並將平台\(窗口孫迪\)提交Chart

* 給孫迪的東西
  * [https://harbor.arfa.wise-paas.com/harbor/projects/79/helm-charts/datahub/versions/2.0.1](https://harbor.arfa.wise-paas.com/harbor/projects/79/helm-charts/datahub/versions/2.0.1)
* 提交給孫迪的chart是位於林口的harbor, 注意是datahub project, 而不是datahub-dev project
  * datahub-dev是開發階段用的
  * datahub才是透過CI產生出來放置的地方
* chart裡會包含worker/portal的image位置與版號
  * 這是透過Step4 CI去version list取得對應的小版號並置換

## 



