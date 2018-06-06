# TunnelBufferedWriter {#concept_ns5_qlg_vdb .concept}

一次完整的上传流程通常包括以下步骤：

1.  先对数据进行划分。
2.  为每个数据块指定block Id，即调用openRecordWriter\(id\)。
3.  用一个或多个线程分别将这些block上传上去，并在某个block上传失败以后，需要对整个block进行重传。
4.  在所有block都上传以后，向服务端提供上传成功的blockid list进行校验，即调用session.commit\(\[1,2,3,…\]\)。

    由于服务端对block管理，连接超时等的一些限制，上传过程逻辑变得比较复杂，为了简化上传过程，SDK提供了更高级的一种RecordWriter—TunnelBufferWriter。


**接口定义如下：**

```
public class TunnelBufferedWriter implements RecordWriter {
        public TunnelBufferedWriter(TableTunnel.UploadSession session, CompressOption option) throws IOException;
        public long getTotalBytes();
        public void setBufferSize(long bufferSize);
        public void setRetryStrategy(RetryStrategy strategy);
        public void write(Record r) throws IOException;
        public void close() throws IOException;
    }
```

**TunnelBufferedWriter 对象：**

-   生命周期：从创建RecordWriter到数据上传结束。
-   创建TunnelBufferedWriter实例：通过调用UploadSession的openBufferedWriter接口创建。
-   数据上传：调用Write接口，数据会先写入本地缓存区，缓存区满后会批量提交到服务端，避免了连接超时，同时，如果上传失败会自动进行重试。
-   结束上传：调用close接口，最后再调用UploadSession的commit接口，即可完成上传。
-   缓冲区控制：可以通过setBufferSize接口修改缓冲区占内存的字节数（bytes），建议设置64M以上的大小，避免服务端产生过多小文件，影响性能，一般无须设置，维持默认值即可。
-   重试策略设置：您可以选择三种重试回避策略：指数回避（EXPONENTIAL\_BACKOFF）、线性时间回避（LINEAR\_BACKOFF）、常数时间回避（CONSTANT\_BACKOFF）。例如：下面这段代码可以将Write的重试次数调整为6，每一次重试之前先分别回避4s、8s、16s、32s、64s和128s（从4开始的指数递增的序列），这个也是默认的行为，一般情况不建议调整。

    ```
    RetryStrategy retry 
      = new RetryStrategy(6, 4, RetryStrategy.BackoffStrategy.EXPONENTIAL_BACKOFF)
    writer = (TunnelBufferedWriter) uploadSession.openBufferedWriter();
    writer.setRetryStrategy(retry);
    ```


