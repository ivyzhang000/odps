# 开发PyOdps脚本 {#concept_ffm_wdd_xdb .concept}

PyOdps是MaxCompute Python版本的SDK， 它提供了对MaxCompute对象的基本操作，并提供了DataFrame框架，您可以在MaxCompute上进行数据分析。

1.  使用PyOdps开发脚本，首先需要[安装PyOdps及Python插件](ZH-CN_TP_13746_V1.dita)，安装完成后即可直接新建一个MaxCompute PyODPS脚本。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13748/3447_zh-CN.png)

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13748/3452_zh-CN.png)

2.  新建成功后，模板会通过PyOdps[room对象](http://pyodps.alibaba.net/pyodps-docs/interactive-zh.html)，自动初始化odps和o这两个对象。

    **说明：** 在云端D2运行时会自动在后台创建，所以为了IDEA编译通过，需要显式创建。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13748/3455_zh-CN.png)


