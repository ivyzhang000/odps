# 编写MapReduce {#concept_blb_kxd_vdb .concept}

本文将为您介绍安装好MaxCompute客户端后，如何快速运行MapReduce WordCount示例程序。

**说明：** 如果您使用Maven，可以从[Maven 库](http://search.maven.org/)中搜索odps-sdk-mapred获取不同版本的Java SDK。相关配置信息如下所示：

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-mapred</artifactId>
    <version>0.26.2-public</version>
</dependency>
```

## 前提条件 {#section_ht1_byd_vdb .section}

-   编译、运行MapReduce时，需要首先安装JDK1.6或以上版本。
-   请参见[安装并配置客户端](../cn.zh-CN/准备工作/安装并配置客户端.md)对MaxCompute客户端进行部署。更多关于MaxCompute客户端的使用，请参见[MaxCompute客户端](../cn.zh-CN/工具及下载/客户端.md)。

## 操作步骤 {#section_mcx_3yd_vdb .section}

1.  安装并配置好客户端后，打开odpscmd.bat，进入相应项目空间中。
2.  输入建表语句，创建输入和输出表。如下所示：

    ```
    CREATE TABLE wc_in (key STRING, value STRING);
    CREATE TABLE wc_out (key STRING, cnt BIGINT);
    -- 创建输入、输出表
    ```

    更多创建表的语句请参见[创建表](../cn.zh-CN/用户指南/常用命令/表操作.md)。

3.  上传数据。

    您可以通过以下两种方式上传数据。

    -   使用Tunnel命令上传数据。

        ```
        tunnel upload kv.txt wc_in
        -- 上传示例数据
        ```

        kv.txt文件中的数据如下所示：

        ```
        238,val_238
        186,val_86
        186,val_86
        ```

    -   您也可以通过SQL语句直接插入数据，示例如下：

        ```
        insert into table wc_in select '238',' val_238' from (select count(*) from wc_in) a;
        ```

4.  编写MapReduce程序并编译。

    MaxCompute为您提供了便捷的Eclipse开发插件，方便您快速开发MapReduce程序，并提供了本地调试MapReduce的功能。

    您需要先在Eclipse中创建一个项目工程，而后在此工程中编写MapReduce程序。本地调试通过后，将编译好的程序（Jar 包，如[Word-count-1.0.jar](../cn.zh-CN/用户指南/MapReduce/示例程序/WordCount示例.md)）导出并上传至MaxCompute。详情请参见[MapReduce开发插件介绍](../cn.zh-CN/工具及下载/Eclipse开发插件/安装Eclipse插件.md)。

5.  添加Jar包到project资源（比如这里的Jar包名为word-count-1.0.jar）。

    ```
    add jar word-count-1.0.jar;
    ```

6.  在MaxCompute客户端运行Jar命令。

    ```
    jar -resources word-count-1.0.jar -classpath /home/resources/word-count-1.0.jar com.taobao.jingfan.WordCount wc_in wc_out;
    ```

7.  在MaxCompute客户端查看结果。

    ```
    select * from wc_out;
    ```

    **说明：** 如果您在 Java 程序中使用了任何资源，请务必将此资源加入-resources参数。Jar命令的详细介绍请参见[作业提交](../cn.zh-CN/用户指南/MapReduce/功能介绍/作业提交.md)*。*


