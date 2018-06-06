# 编写Graph {#concept_gzg_1c2_vdb .concept}

本文将以[SSSP算法](../cn.zh-CN/用户指南/图模型/示例程序/单源最短距离.md)为例，为您介绍如何提交Graph作业。

[Graph](../cn.zh-CN/用户指南/图模型/图模型概述.md)作业的提交方式与[MapReduce](../cn.zh-CN/用户指南/MapReduce/功能介绍/作业提交.md)基本相同。如果您使用Maven，可以从[Maven库](http://search.maven.org/)中搜索`odps-sdk-graph`获取不同版本的Java SDK，相关配置信息如下所示：

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-graph</artifactId>
    <version>0.20.7</version>
</dependency>
```

## 操作步骤 {#section_esm_lc2_vdb .section}

1.  进入console并运行odpscmd。
2.  创建输入表和输出表 。

    ```
    create table sssp_in (v bigint, es string);
    create table sssp_out (v bigint, l bigint);
    ```

    创建表的更多语句请参见[DDL语句](../cn.zh-CN/用户指南/SQL/DDL语句.md)。

3.  上传数据。

    本地数据的内容如下：

    ```
    1 2:2,3:1,4:4
    2 1:2,3:2,4:1
    3 1:1,2:2,5:1
    4 1:4,2:1,5:1
    5 3:1,4:1
    ```

    以空格键做两列的分隔符，执行Tunnel命令上传数据。

    ```
    tunnel u -fd " " sssp.txt sssp_in;
    ```

4.  编写SSSP示例。

    根据[Graph开发插件](../cn.zh-CN/工具及下载/Eclipse开发插件/Graph开发插件介绍.md)的介绍，本地编译、调试[SSSP算法示例](../cn.zh-CN/用户指南/图模型/示例程序/单源最短距离.md)。本示例中假设代码被打包为odps-graph-example-sssp.jar。

    **说明：** 仅需要将SSSP代码打包即可，不需要同时将SDK打入odps-graph-example-sssp.jar中。

5.  添加Jar资源。

    ```
    add jar $LOCAL_JAR_PATH/odps-graph-example-sssp.jar
    ```

    **说明：** 创建资源的介绍请参见[资源操作](../cn.zh-CN/用户指南/常用命令/资源操作.md)。

6.  运行SSSP。

    ```
    jar -libjars odps-graph-example-sssp.jar -classpath $LOCAL_JAR_PATH/odps-graph-example-sssp.jar com.aliyun.odps.graph.example.SSSP 1 sssp_in sssp_out;
    ```

    Jar命令用于运行MaxCompute Graph作业，用法与[MapReduce](../cn.zh-CN/用户指南/MapReduce/功能介绍/作业提交.md)作业的运行命令完全一致。

    Graph作业执行时命令行会打印作业实例ID，执行进度，结果Summary等。

    输出示例如下所示：

    ```
    ID = 20130730160742915gl205u3
    2013-07-31 00:18:36     SUCCESS
    Summary:
    Graph Input/Output
    Total input bytes=211
    Total input records=5
    Total output bytes=161
    Total output records=5
    graph_input_[bsp.sssp_in]_bytes=211
    graph_input_[bsp.sssp_in]_records=5
    graph_output_[bsp.sssp_out]_bytes=161
    graph_output_[bsp.sssp_out]_records=5
    Graph Statistics
    Total edges=14
    Total halted vertices=5
    Total sent messages=28
    Total supersteps=4
    Total vertices=5
    Total workers=1
    Graph Timers
    Average superstep time (milliseconds)=7
    Load time (milliseconds)=8
    Max superstep time (milliseconds) =14
    Max time superstep=0
    Min superstep time (milliseconds)=5
    Min time superstep=2
    Setup time (milliseconds)=277
    Shutdown time (milliseconds)=20
    Total superstep time (milliseconds)=30
    Total time (milliseconds)=344
    OK
    ```

    **说明：** 如果您需要使用Graph功能，直接开通提交图计算作业即可。


