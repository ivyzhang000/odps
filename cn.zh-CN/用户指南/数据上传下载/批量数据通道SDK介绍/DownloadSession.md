# DownloadSession {#concept_yvg_zkg_vdb .concept}

**接口定义如下：**

```
public class DownloadSession {
        DownloadSession(Configuration conf, String projectName, String tableName,
            String partitionSpec) throws TunnelException
        DownloadSession(Configuration conf, String projectName, String tableName,
            String partitionSpec, String downloadId) throws TunnelException
        public String getId()
        public long getRecordCount()
        public TableSchema getSchema()
        public DownloadSession.Status getStatus()
        public RecordReader openRecordReader(long start, long count)
        public RecordReader openRecordReader(long start, long count, boolean compress)
    }
```

**接口说明如下：**

-   **生命周期**：从创建Download实例到下载结束。
-   **创建Download实例**：您可以通过**调用构造方法**和TableTunnel两种方式进行创建。
    -   请求方式：同步。
    -   Server端会为该Download创建一个Session，生成唯一DownloadId标识该Download，客户端可以通过getId获取。
    -   该操作开销较大，Server端会对数据文件创建索引，当文件数很多时，该时间会比较长。
    -   同时Server端会返回总Record数，可以根据总Record数启动多个并发同时下载。
-   **下载数据**：
    -   请求方式：异步。
    -   调用openRecordReader方法，生成RecordReader实例，其中参数start标识本次下载的Record的起始位置，从0开始，取值范围是\>= 0， count标识本次下载的记录数，取值范围是\>0。
-   **查看下载**：
    -   请求方式：同步。
    -   调用getStatus，可以获取当前Download状态。
-   四种状态说明如下：
    -   UNKNOWN：Server端刚创建一个Session时设置的初始值。
    -   NORMAL：创建Download对象成功。
    -   CLOSED：下载结束后。
    -   EXPIRED：下载超时。

