# TableTunnel {#concept_exm_f3g_vdb .concept}

TableTunnel是访问MaxCompute Tunnel服务的入口类，TableTunnel.UploadSession接口表示一个向MaxCompute表中上传数据的会话，TableTunnel.DownloadSession表示一个向MaxCompute表中下载数据的会话。

**接口定义如下：**

```
public class TableTunnel {
 public DownloadSession createDownloadSession(String projectName, String tableName);
 public DownloadSession createDownloadSession(String projectName, String tableName, PartitionSpec partitionSpec);
 public UploadSession createUploadSession(String projectName, String tableName);
 public UploadSession createUploadSession(String projectName, String tableName, PartitionSpec partitionSpec);
 public DownloadSession getDownloadSession(String projectName, String tableName, PartitionSpec partitionSpec, String id);
 public DownloadSession getDownloadSession(String projectName, String tableName, String id);
 public UploadSession getUploadSession(String projectName, String tableName, PartitionSpec partitionSpec, String id);
 public UploadSession getUploadSession(String projectName, String tableName, String id);
 }
```

**接口说明如下：**

-   生命周期：从TableTunnel实例被创建开始，一直到程序结束。
-   提供创建Upload对象和Download对象的方法。
-   对一张表或partition上传下载的过程，称为一个session。session由一或多个到Tunnel RESTful API的HTTP Request组成。
-   TableTunnel的上传session是INSERT INTO语义，即对同一张表或partition的多个/多次上传session互不影响，每个session上传的数据会位于不同的目录中。
-   在上传session中，每个RecordWriter对应一个HTTP Request，由一个block id标识，对应service端一个文件（实际上block id就是对应的文件名）。
-   同一session中，使用同一block id多次打开RecordWriter会导致覆盖行为，最后一个调用close\(\)的RecordWriter所上传的数据会被保留。该特性可用于block的上传失败重传。

**TableTunnel接口实现流程：**

1.  RecordWriter.write\(\)将数据上传到临时目录的文件。
2.  RecordWriter.close\(\)将相应的文件从临时目录挪移到data目录。
3.  session.commit\(\)将相应data目录下的所有文件挪移到相应表所在目录，并更新表meta，即数据进表，对其他MaxCompute任务（如SQL、MR）可见。

**TableTunnel接口限制：**

-   block id 的取值范围是\[0, 20000\)，单个block上传的数据限制为100G。
-   session的超时时间为24小时。如果，大批量数据传送导致超过24小时，需要自行拆分成多个session。
-   RecordWriter对应的HTTP Request超时为120s。如果120s内HTTP连接上没有数据流过，service端会主动关闭连接。

    **说明：** HTTP本身还有8k buffer，因此并不是每次调用RecordWriter.write\(\)都能保证HTTP连接上有数据流过。TunnelRecordWriter.flush\(\)可以将buffer内数据强制刷出。

-   对于日志类写入MaxCompute的场景，因为数据到达不可预测，容易造成RecordWriter超时。此时：
    -   不建议对每条数据打开一个RecordWriter（因为每个RecordWriter对应一个文件，会造成小文件过多，严重影响MaxCompute后续使用性能）。
    -   建议用户代码cache至少64M的数据再使用一个RecordWriter一次性批量写入。
-   RecordReader的超时为300s。

