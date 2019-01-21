# WISE-PaaS SCADA

---

資料獲取與監視控制系統 \(SCADA\) 是一款自動化 IoT 應用軟硬體元件管理系統，專門用於資料獲取、集成、分析和視覺化技術。

* [User Guide](https://portal-technical.wise-paas.com/doc/document-portal.html#SCADA-1)



### 系統架構

* 整個系統由Node.js所開發, 分成以下App和一些共用模組\(node\_module\)
  * SCADA DataWorker: 專門處理Edge端設備狀態管理以及所上傳的資料, 根據資料類型寫入不同的的資料庫. 目前支援有PostgreSQL, MongoDB, 和InfluxDB.
  * SCADA Portal: 提供介面讓使用者進行設備管理和察看數據, 也提供Resuful API供其它SRP做二次開發.
  * 共用模組\(node\_module\)
    * wise-paas-scada-dbmigrate: 負責處理PostgreSQL資料庫版本管理, 使用[Flyway](https://flywaydb.org/).
    * wise-paas-common-utility: 因為許多共用模組都會使用到AMQP和MongoDB連線, 所以再拆分成此common模組.
    * wise-paas-scada-utility: 主要處理與MongoDB有關的共用功能.
    * wise-paas-scada-dbmanager: 主要處理與PostgreSQL有關的共用功能.





