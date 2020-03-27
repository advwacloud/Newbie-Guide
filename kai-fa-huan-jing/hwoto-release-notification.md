# Release Notification教學

**Release前請確認相關資訊已經更新, 可以看前一次release 的git log怎麼改**

---

1. 準備notification release note草稿給和益
2. notification打TAG, 觸發CI自動建立/推送image到林口harbor \(notification project\)
3. 在chart專案(pipeline_Notification_helmChart_Build)也打TAG, 觸發CI自動建立/推送chart到林口harbor \(notification project\)
4. image/chart都建好後, 通知和益
5. 和益會發正式Release Note, 並向平台\(窗口孫迪\)提交Chart

---

## Step1. 準備notification release note草稿給和益

* 慣例是會拿專案裡的CHANGELOG.md稍作整理後傳給和益做最後修改
  * 例如一些內部的修改, 會是備註類文字的可以拿掉

## Step2. notification打TAG, 觸發CI自動建立/推送image到林口harbor \(notification project\)

* notification release時, 都會merge回master branch, 並打上Tag
* 我們有串jenkins CI PIpeline, 會自動build image, 並傳到林口harbor下的notification project

## Step3. 在chart專案(pipeline_Notification_helmChart_Build)也打TAG, 觸發CI自動建立/推送chart到林口harbor \(notification project\)

* http://advgitlab.eastasia.cloudapp.azure.com/DataHub/pipeline_Notification_helmChart_Build

* 每個notification release會搭配一個chart

* 所以要clone helm chart專案, 並打上tag

* helm chart不一定每次release都會修改, 但還是要build一個對應版號的chart

  * 也就是說, 同一個commit可能會打多個Tag

## Step4. image/chart都建好後, 通知和益

跟和益說image/chart都建好了

## Step5. 和益會發正式Release Note, 並將平台\(窗口孫迪\)提交Chart

* 給孫迪的東西
  * https://harbor.arfa.wise-paas.com/harbor/projects/18/helm-charts/notification/versions/1.1.0
* 提交給孫迪的chart是位於林口的harbor, 注意是notificationproject, 而不是notification-dev project
  * notification-dev是開發階段用的
  * notification才是透過CI產生出來放置的地方
* chart裡會包含notification的image位置與版號




