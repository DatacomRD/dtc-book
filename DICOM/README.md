> dtc-dicom 並未公開 release，若有使用需求請透過[合華官網][datacom]聯繫

dtc-dicom 是將 [dcm4che] 封裝成更方便使用的 library


### Maven ###

```XML
<dependency>
	<groupId>com.dtc.common</groupId>
	<artifactId>dicom</artifactId>
	<version>0.0.2-SNAPSHOT</version>
</dependency>
```

SCP
===
* C-STORE：`StoreService`


SCU
===

* C-FIND：`Finder`、`MwlFinder`
* C-GET：`Getter`
* C-STORE：`Storer`


Utility
=======

* `AttributeUtl`：將 DICOM 轉成 [dcm4che] `Attributes` 的相關 util
* `DicomExport`：將 DICOM 檔 unmarshall 成指定 Java class 的 instance
* `Workflow` & `Task`：以 task 方式定義一個 DICOM 檔的處理流程
	* 參見 `Task` 的實作 class
* `FlowManager`：在不產生 OOM 的前提下盡可能同時處理多個 DICOM 檔



[datacom]: http://www.datacom.com.tw/
[dcm4che]: http://www.dcm4che.org/