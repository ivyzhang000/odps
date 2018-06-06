# 导出SQL的运行结果 {#concept_bsq_ctc_5db .concept}

本文将通过示例，为您介绍几种下载 MaxCompute SQL 计算结果的方法。

**说明：** 本文中所有的 SDK 部分仅举例介绍 Java 的例子。

您可以通过以下几种方法导出SQL的运行结果：

-   如果数据比较少，可以直接用[SQL Task](../cn.zh-CN/用户指南/SDK/Java SDK.md)得到全部的查询结果。

-   如果只是想导出某个表或者分区，可以用 [Tunnel](../cn.zh-CN/用户指南/数据上传下载/Tunnel命令操作.md)直接导出数据。

-   如果 SQL 比较复杂，需要 Tunnel 和 SQL 相互配合才行。

-   [DataWorks](https://data.aliyun.com/product/ide?) 可以方便地帮您运行 SQL，[同步数据](https://help.aliyun.com/document_detail/47677.html)，并有定时调度，配置任务依赖的功能。

-   开源工具 **DataX** 可帮助您方便地把 MaxCompute 中的数据导出到目标数据源，详情请参见 [DataX 概述](https://help.aliyun.com/document_detail/28291.html)。


## SQLTask 方式导出 {#section_pyd_ntc_5db .section}

[SQL Task](../cn.zh-CN/用户指南/SDK/Java SDK.md) 是 SDK 直接调用 MaxCompute SQL 的接口，能很方便地运行 SQL 并获得其返回结果。

从文档可以看到，`SQLTask.getResult(i);` 返回的是一个 List，可以循环迭代这个 List，获得完整的 SQL 计算返回结果。不过此方法有个缺陷，详情请参见 [其他操作](../cn.zh-CN/用户指南/常用命令/其他操作.md)中提到的`SetProject READ_TABLE_MAX_ROW`功能。

目前 Select 语句返回给客户端的数据条数最大可以调整到 **1万**。也就是说如果在客户端上（包括 SQLTask）直接 Select，那相当于查询结果上最后加了个 Limit N（如果是 CREATE TABLE XX AS SELECT 或者用 INSERT INTO/OVERWRITE TABLE 把结果固化到具体的表里就没关系）。

## Tunnel 方式导出 {#section_ryd_ntc_5db .section}

如果您需要导出的查询结果是某张表的全部内容（或者是具体的某个分区的全部内容），可以通过 Tunnel 来实现，详情请参见 [命令行工具](../cn.zh-CN/用户指南/数据上传下载/Tunnel命令操作.md) 和基于 SDK 编写的 [Tunnel SDK](../cn.zh-CN/用户指南/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md)。

在此提供一个 Tunnel 命令行导出数据的简单示例，Tunnel SDK 的编写是在有一些命令行没办法支持的情况下才需要考虑，详情请参见 [批量数据通道概述](../cn.zh-CN/用户指南/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md)。

```

tunnel d wc_out c:\wc_out.dat;
2016-12-16 19:32:08 - new session: 201612161932082d3c9b0a012f68e7 total lines: 3
2016-12-16 19:32:08 - file [0]: [0, 3), c:\wc_out.dat
downloading 3 records into 1 file
2016-12-16 19:32:08 - file [0] start
2016-12-16 19:32:08 - file [0] OK. total: 21 bytes
download OK
```

## SQLTask+Tunnel 方式导出 {#section_vyd_ntc_5db .section}

从前面 SQL Task 方式导出的介绍可以看到，SQL Task 不能处理超过 1 万条记录，而 Tunnel 可以，两者可以互补。所以可以基于两者实现数据的导出。

代码实现的示例如下：

```

private static final String accessId = "userAccessId";
private static final String accessKey = "userAccessKey";
private static final String endPoint = "http://service.odps.aliyun.com/api";
private static final String project = "userProject";
private static final String sql = "userSQL";
private static final String table = "Tmp_" + UUID.randomUUID().toString().replace("-", "_");//其实也就是随便找了个随机字符串作为临时表的名字
private static final Odps odps = getOdps();
public static void main(String[] args) {
System.out.println(table);
runSql();
tunnel();
}
/*
* 把SQLTask的结果下载过来
* */
private static void tunnel() {
TableTunnel tunnel = new TableTunnel(odps);
try {
DownloadSession downloadSession = tunnel.createDownloadSession(
project, table);
System.out.println("Session Status is : "
+ downloadSession.getStatus().toString());
long count = downloadSession.getRecordCount();
System.out.println("RecordCount is: " + count);
RecordReader recordReader = downloadSession.openRecordReader(0,
count);
Record record;
while ((record = recordReader.read()) != null) {
consumeRecord(record, downloadSession.getSchema());
}
recordReader.close();
} catch (TunnelException e) {
e.printStackTrace();
} catch (IOException e1) {
e1.printStackTrace();
}
}
/*
* 保存这条数据
* 数据量少的话直接打印后拷贝走也是一种取巧的方法。实际场景可以用Java.io写到本地文件，或者写到远端数据等各种目标保存起来。
* */
private static void consumeRecord(Record record, TableSchema schema) {
System.out.println(record.getString("username")+","+record.getBigint("cnt"));
}
/*
* 运行SQL，把查询结果保存成临时表，方便后面用Tunnel下载
* 这里保存数据的lifecycle为1天，所以哪怕删除步骤出了问题，也不会太浪费存储空间
* */
private static void runSql() {
Instance i;
StringBuilder sb = new StringBuilder("Create Table ").append(table)
.append(" lifecycle 1 as ").append(sql);
try {
System.out.println(sb.toString());
i = SQLTask.run(getOdps(), sb.toString());
i.waitForSuccess();
} catch (OdpsException e) {
e.printStackTrace();
}
}
/*
* 初始化MaxCompute(原ODPS)的连接信息
* */
private static Odps getOdps() {
Account account = new AliyunAccount(accessId, accessKey);
Odps odps = new Odps(account);
odps.setEndpoint(endPoint);
odps.setDefaultProject(project);
return odps;
}
```

## 大数据开发套件的数据同步方式导出 {#section_d12_ntc_5db .section}

前面介绍的方式解决了数据下载后保存的问题，但是没解决数据的生成以及两个步骤之间的调度依赖的问题。

[数加·DataWorks](https://data.aliyun.com/product/ide?) 可以运行 SQL、[配置数据同步任务](https://help.aliyun.com/document_detail/30269.html)，还可以设置自动 [周期性运行](https://help.aliyun.com/document_detail/50130.html) 和 [多任务之间依赖](https://help.aliyun.com/document_detail/50130.html)，彻底解决了前面的烦恼。

接下来将用一个简单示例，为您介绍如何通过大数据开发套件运行 SQL 并配置数据同步任务，以完成数据生成和导出需求。

**操作步骤**

1.  创建一个工作流，工作流里创建一个 SQL 节点和一个数据同步节点，并将两个节点连线配置成依赖关系，SQL 节点作为数据产出的节点，数据同步节点作为数据导出节点。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1131_zh-CN.png)

2.  配置 SQL 节点。

    **说明：** SQL 这里的创建表要先执行一次再去配置同步（否则表都没有，同步任务没办法配置）。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1133_zh-CN.png)

3.  配置数据同步任务。

    1.  选择来源。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1138_zh-CN.png)

    2.  选择目标。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1139_zh-CN.png)

    3.  字段映射。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1145_zh-CN.png)

    4.  通道控制。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1146_zh-CN.png)

    5.  预览保存。

4.  工作流调度配置完成后（可以直接使用默认配置），保存并提交工作流，然后单击 **测试运行**。查看数据同步的运行日志，如下所示：

    ```
    
    2016-12-17 23:43:46.394 [job-15598025] INFO JobContainer - 
    任务启动时刻 : 2016-12-17 23:43:34
    任务结束时刻 : 2016-12-17 23:43:46
    任务总计耗时 : 11s
    任务平均流量 : 31.36KB/s
    记录写入速度 : 1668rec/s
    读出记录总数 : 16689
    读写失败总数 : 0
    ```

5.  输入 SQL 语句查看数据同步的结果，如下图所示：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12158/1148_zh-CN.png)


