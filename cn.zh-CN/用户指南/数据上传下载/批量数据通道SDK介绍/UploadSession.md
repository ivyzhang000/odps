# UploadSession {#concept_ecn_pjg_vdb .concept}

**接口定义如下：**

```
public class UploadSession {
        UploadSession(Configuration conf, String projectName, String tableName,
            String partitionSpec) throws TunnelException;
        UploadSession(Configuration conf, String projectName, String tableName, 
            String partitionSpec, String uploadId) throws TunnelException;
        public void commit(Long[] blocks);
        public Long[] getBlockList();
        public String getId();
        public TableSchema getSchema();
        public UploadSession.Status getStatus();
        public Record newRecord();
        public RecordWriter openRecordWriter(long blockId);
        public RecordWriter openRecordWriter(long blockId, boolean compress);
        public RecordWriter openBufferedWriter();
        public RecordWriter openBufferedWriter(boolean compress);
    }
```

**接口说明如下：**

-   **生命周期**：从创建Upload实例到结束上传。
-   **创建Upload实例**：您可以通过**调用构造方法**和**TableTunnel**两种方式进行创建。
    -   请求方式：同步。
    -   Server端会为该Upload创建一个session， 生成唯一UploadId标识该Upload，客户端可以通过getId获取。
-   **上传数据**：
    -   请求方式：同步。
    -   调用openRecordWriter方法，生成RecordWriter实例，其中参数blockId用于标识此次上传的数据，也描述了数据在整个表中的位置，取值范围为\[0,20000\]，当数据上传失败，可以根据blockId重新上传。
-   **查看上传**：
    -   请求方式：同步。
    -   调用getStatus可以获取当前Upload状态。
    -   调用getBlockList可以获取成功上传的blockid list，可以和上传的blockid list对比，对失败的blockId重新上传。
-   **结束上传**：
    -   请求方式：同步。
    -   调用commit（Long\[\] blocks）方法，参数blocks列表表示已经成功上传的block列表，Server端会对该列表进行验证。
    -   该功能是加强对数据正确性的校验，如果提供的block列表与Server端存在的block列表不一致抛出异常。
    -   Commit失败可以进行重试。
-   七种状态说明如下：

-   UNKNOWN：Server端刚创建一个Session时设置的初始值。
-   NORMAL：创建Upload对象成功。
-   CLOSING：当调用complete方法（结束上传）时，服务端会先把状态置为CLOSING。
-   CLOSED：完成结束上传（即把数据移动到结果表所在目录）后。
-   EXPIRED：上传超时。
-   CRITICAL：服务出错。

**说明：** 

-   同一个Upload中Session的blockId不能重复。也就是说，对于同一个UploadSession，用一个blockId打开RecordWriter，写入一批数据后，调用close，然后再commit完成后，不可以重新再用该blockId打开另一个RecordWriter写入数据。
-   一个block大小上限100GB，建议大于64M的数据。
-   每个Session在服务端的生命周期为24小时。
-   上传数据时，Writer每写入8KB数据，便会触发一次网络动作，如果120秒内没有网络动作，服务端将主动关闭连接，此时Writer将不可用，请重新打开一个新的Writer写入。
-   建议使用openBufferedWriter接口上传数据，该接口对您屏蔽了blockId的细节，并且内部带有数据缓存区，会自动进行失败重试，详情请参见TunnelBufferedWriter的介绍和示例。

