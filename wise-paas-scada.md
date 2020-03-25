# WISE-PaaS DataHub

---

WISE-PaaS/DataHub 是針對時間序列 \(Time-series\) 數據的處理服務，提供了數據收集 \(Data Collection\)，數據聚合 \(Data Aggregation\), 數據監控 \(Data Monitoring\) 和警報通知 \(Alert Notification\) 功能。透過 WISE-PaaS/DataHub 能夠輕鬆建立出基於時間序列數據的應用服務。WISE-PaaS/DataHub 可以對應各種 IoT 設備，閘道器和系統等產生的大量時間序列數據持續不斷地收集，處理和儲存。我們提供了六種主流語言的 Edge SDK 供使用者整合，可以應用於各種作業系統和硬體環境。另外使用者也可以透過 WISE-PaaS/DataHub API 來處理寫入到 WISE-PaaS/DataHub 上的數據, 例如：即時數據，歷史數據，統計數據，警報訊息等。

* [User Guide](https://docs.wise-paas.advantech.com/en/Guides_and_API_References/Data_Acquisition/DataHub)

### 系統架構

* 整個系統由Node.js所開發, 分成以下App和一些共用模組\(node\_module\)
  * SCADA DataWorker: 專門處理Edge端設備狀態管理以及所上傳的資料, 根據資料類型寫入不同的的資料庫. 目前支援有PostgreSQL, MongoDB, 和InfluxDB.
  * SCADA Portal: 提供介面讓使用者進行設備管理和察看數據, 也提供Resuful API供其它SRP做二次開發.
  * 共用模組\(node\_module\)
    * wise-paas-scada-dbmigrate: 負責處理PostgreSQL資料庫版本管理, 使用[Flyway](https://flywaydb.org/).
    * wise-paas-common-utility: 因為許多共用模組都會使用到AMQP和MongoDB連線, 所以再拆分成此common模組.
    * wise-paas-scada-utility: 主要處理與MongoDB有關的共用功能.
    * wise-paas-scada-dbmanager: 主要處理與PostgreSQL有關的共用功能.



