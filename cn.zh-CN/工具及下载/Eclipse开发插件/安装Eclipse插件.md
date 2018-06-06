# 安装Eclipse插件 {#concept_p3g_jp5_vdb .concept}

为了方便您使用[MapReduce](../DNODPS1898901/ZH-CN_TP_12013_V1.dita) 及 [UDF](../DNODPS1898901/ZH-CN_TP_12002_V1.dita) 的 Java SDK 进行开发工作，MaxCompute（原 ODPS） 提供了 Eclipse 开发插件。该插件能够模拟 MapReduce 及 UDF 的运行过程，为您提供本地调试手段，并提供了简单的模板生成功能。

**说明：** 

-   目前高版本的 Eclipse Neon 有可能会导致插件加载失败，请使用 Eclipse Luna 版本。

-   请单击 [此处](https://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/cn/odps/0.0.90/assets/download/odps-eclipse-plugin-bundle-0.16.0.zip) 下载插件。

-   与 MapReduce 提供的本地运行模式不同，Eclipse 插件不能够与 MaxCompute 同步数据。您使用的数据需要手动拷贝到 Eclipse 插件的 warehouse 目录下。


下载 Eclipse 插件后，将软件包解压，会看到如下 Jar 内容：

```
odps-eclipse-plugin-bundle-0.16.0.jar
```

将插件放置在 Eclipse 安装目录的plugins子目录下。打开 Eclipse，单击右上角的打开透视图（Open Perspective）：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12150/2933_zh-CN.png)

单击后出现下面的对话框：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12150/2934_zh-CN.png)

选择 ODPS，单击**OK** 。同样在右上角会出现 ODPS 图标，表示插件生效：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12150/2935_zh-CN.png)

