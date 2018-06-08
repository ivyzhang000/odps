# 开发MapReduce {#concept_nvh_s2c_5db .concept}

创建完成[MaxCompute Java Module](cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/创建 MaxCompute Java Module.md)后，即可以开始开发[MR](../cn.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md)了。

## 开发MR {#section_gr4_mgg_vdb .section}

1.  在module的源码目录即src-\>main上右键**new**，选择**MaxCompute Java**。

2.  分别创建Driver，Mapper，Reducer。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/1997_zh-CN.png)

3.  模板已自动填充框架代码，只需要设置输入/输出表，Mapper/Reducer类等即可。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/1998_zh-CN.png)


## 调试MR {#section_jr4_mgg_vdb .section}

MR开发好后，下一步就是要测试自己的代码，看是否符合预期，我们支持两种方式:

**单元测试**：在examples目录下有WordCount的单测实例，可参考例子编写自己的UT。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/1999_zh-CN.png)

**本地运行MR**：本地运行时，需要指定运行数据源，有两种方式设定测试数据源：

-   studio通过tunnel服务自动下载指定MaxCompute project的表数据到warehouse目录下。默认下载100条，如需更多数据测试，请自行使用console的tunnel命令或者studio的表下载功能。

-   提供mock项目\(example\_project\)及表数据，用户可参考warehouse下example\_project自行设置。


1.  运行MR: 在Driver类上右键，点击**运行**菜单，弹出run configuration对话框，配置MR需要在哪个MaxCompute Project上运行即可。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/2001_zh-CN.png)

2.  点击**OK**，如果指定MaxCompute project的表数据未被下载到warehouse中，则首先下载数据；如果采用mock项目或已被下载则跳过。接下来，MR local run框架会读取warehouse中指定表的数据作为MR的输入，开始本地运行MR，用户可以在控制台看到日志输出和结果打印。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/2002_zh-CN.png)


## 生产运行MR {#section_or4_mgg_vdb .section}

本地调试通过后，接下来就可以把MR发布到服务端，在MaxCompute分布式环境下运行了:

1.  首先，你得将自己的MR程序打成jar包，并发布到服务端。[如何打包发布？](https://help.aliyun.com/document_detail/50904.html)

2.  通过studio无缝集成的MaxCompute console（具体的，在Project Explorer Window的project上右键，选择Open in Console），在console命令行中输入类似如下的 [jar命令](../cn.zh-CN/用户指南/MapReduce/功能介绍/作业提交.md)：

    ```
    jar-libjars wordcount.jar -classpath D:\odps\clt\wordcount.jar com.aliyun.odps.examples.mr.WordCount wc_in wc_out;
    ```


