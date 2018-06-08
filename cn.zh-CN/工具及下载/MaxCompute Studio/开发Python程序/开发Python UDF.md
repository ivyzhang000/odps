# 开发Python UDF {#concept_syd_5dd_xdb .concept}

MaxCompute Studio支持Python UDF开发，但需要您根据[Python开发使用须知](cn.zh-CN/工具及下载/MaxCompute Studio/开发Python程序/Python开发使用须知.md)做好准备工作。

## 开发 {#section_b3t_fld_xdb .section}

1.  右键单击**New** \> **MaxCompute Python**。

    **说明：** 如果没有MaxCompute Python选项说明没有Python插件，请确认是否安装成功。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3401_zh-CN.png)

2.  输入类名，如hello，选择类型，此处选择**Python UDF**。填写完成后单击**OK**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3402_zh-CN.png)

3.  模板已自动填充框架代码，您只需要编写UDF的入参出参，以及函数逻辑。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3403_zh-CN.png)


## 测试 {#section_fzk_lmd_xdb .section}

UDF开发完成后，需要测试自己的代码，看是否符合预期。我们支持下载表的部分sample数据到本地运行，进行DeBug，操作如下：

1.  右键单击Editor中的UDF类，单击**运行**，弹出Run/Debug Configurations对话框。

    **说明：** UDF|UDAF|UDTF一般作用于Select子句中表的某些列，此处需配置MaxCompute project、table和columns，**元数据来源于project explorer窗口和warehouse下的Mock项目**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3406_zh-CN.png)

2.  单击**OK**后，通过Tunnel自动下载您指定表的sample数据到本地warehouse目录。

    **说明：** 

    -   如果已经下载过，则不会再次重复下载，否则利用Tunnel服务下载数据。
    -   默认下载100条，如果需要更多数据测试，请自行使用console的Tunnel命令或者Studio的表下载功能。
3.  下载完成后，您可以在warehouse目录看到下载的sample数据。您也可以使用Mock data（即warehouse中的数据自己mock，详情请参见[开发和调试UDF](cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/开发和调试UDF.md)中的本地运行的warehouse目录模块。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3411_zh-CN.png)

4.  本地运行框架会根据您指定的列，获取data文件中指定列的数据，调用UDF本地运行。

    **说明：** 本地运行是通过Pyodps的pyou脚本实现的，命令如`pyou hello.Plus<data`。安装完pyodps后可以使用相应的命令检查该脚本是否存在。

    -   如果您是windows系统，请运行`${python}/../Scripts/pyou`。
    -   如果您是mac系统，请运行`${python}/../pyou`。
    您可以在控制台看到结果打印，也可以在UDF上打断点调试。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3412_zh-CN.png)


## 注册发布 {#section_dy5_trd_xdb .section}

Python UDF测试通过后，即可注册发布到生产上进行使用。**Add Resource**后，**Create Function**即可。详情请参见[打包/上传/注册](cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/打包/上传/注册.md)。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13747/3413_zh-CN.png)

