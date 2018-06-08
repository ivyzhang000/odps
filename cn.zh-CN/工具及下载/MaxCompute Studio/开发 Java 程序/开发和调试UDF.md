# 开发和调试UDF {#concept_nvh_s2b_5db .concept}

创建完成 [MaxCompute Java Module](cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/创建 MaxCompute Java Module.md)后，即可开发 UDF。

## 操作步骤 {#section_aqr_m1g_vdb .section}

1.  展开已创建的 MaxCompute Java Module 目录，导航至**src** \> **main** \> **java** \> **new** ，单击**MaxCompute Java** 。如下图所示：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1944_zh-CN.png)

2.  填写**name**和**kind**，单击**OK** 。如下图所示：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1947_zh-CN.png)

    -   Name：填写创建的 MaxCompute Java Class 名称，如果还没创建package，可以在此处填写 packagename.classname，会自动生成package。

    -   Kind：选择类型。目前支持的类型有：自定义函数（UDF/UDAF/UDTF）、MapReduce（Driver/Mapper/Reducer）、非结构化开发（StorageHandler/Extractor）等。

3.  创建成功后，即可进行开发、编辑、测试 Java 程序。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1948_zh-CN.png)

    **说明：** 这里代码模板可在Intellij中自定义，具体的：Preference - Editor - File and Code Templates，然后在Code标签页中寻找MaxCompute对应的模板修改。


## 调试UDF {#section_vqx_3cg_vdb .section}

开发 UDF 完成后，即可通过单元测试和本地运行两种方式进行测试，看是否符合预期。

**单元测试**

在 examples 目录下有各种类型的单元测试示例，可参考示例编写自己的 Unit Test。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1949_zh-CN.png)

**本地运行**

本地运行 UDF 时，需要指定运行数据源，有以下两种方式设定测试数据源：

-   MaxCompute Studio 通过 Tunnel 服务自动下载指定项目下的表数据到 warehouse 目录下。

-   提供 Mock 项目及表数据，您可参考 warehouse 下的 example\_project 自行设置。


**操作步骤**

1.  右击 UDF 类，单击 **运行**，弹出 run configuration 对话框。UDF/UDAF/UDTF 一般作用于 select 子句中表的某些列，需要配置 MaxCompute project，table 和 column（**元数据来源于 project explorer 和 warehouse 下的 Mock 项目**）。复杂类型的调试也是支持的，如下图：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1950_zh-CN.png)

2.  单击 **OK**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1951_zh-CN.png)

    **说明：** 

    -   如果指定项目下的表数据未被下载到 warehourse 中，需要先下载数据，默认下载100条。默认下载100条，如需更多数据，可配置Download record limit项。
    -   如果采用 Mock 项目或已下载数据，则直接运行。
    -   UDF 的 local run 框架会将 warehouse 中指定列的数据作为 UDF 的输入，开始本地运行 UDF，您可以在控制台看到日志输出和结果打印。

**本地warehouse目录**

本地 warehouse 用于存储表（包括 meta 和数据）或资源，用于本地执行 UDF 或 MR。warehouse 目录如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12130/1952_zh-CN.png)

**说明：** 

-   warehouse 目录下依次是项目名，tables，表名，表 schema 和 sample data。
-   schema 文件依次配置项目名，表名，以及列名和类型（冒号分隔），分区表还需配置分区列（非分区表参考 wc\_in1，分区表参考 wc\_in2）。
-   data 文件采用标准 csv 格式存储表的 sample 数据：
    -   特殊字符为逗号，双引号和换行（\\n 或 \\r\\n）
    -   列分隔符为逗号，行分隔符为 \\n 或 \\r\\n
    -   如果列内容里包含特殊字符，需要在该列内容前后加上双引号，例如：3,No -\> “3, No”
    -   如果列内容包含双引号，则每个双引号转义成两个双引号，例如：a”b”c -\> “a””b””c”
    -   \\N 表示该列为 null，如果该列内容（string 类型）就是 \\N，需要转换为 “””\\N”””
    -   文件字符编码为UTF-8

