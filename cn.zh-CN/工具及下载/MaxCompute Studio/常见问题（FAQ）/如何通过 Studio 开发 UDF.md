# 如何通过 Studio 开发 UDF {#concept_fmq_1n4_vdb .concept}

本文介绍如何通过studio来开发MaxCompute Java UDF。

1.  **新增一个module:** 参考[创建MaxCompute java module](cn.zh-CN/工具及下载/MaxCompute Studio/开发 SQL 程序/创建 MaxCompute Script Module.md#)，我们的UDF代码将放置在此module中。
2.  **开发UDF:** 参考[UDF开发](cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/开发和调试UDF.md#) studio提供了UDF开发模板，你可以依据模板完成UDF开发。
3.  **测试:** studio提供了UDF本地调试的机制，你可以参考UDF test模板编写自己的测试用例。
4.  **打包:** 你可以依赖IDE本身提供的打包功能，将自己的UDF源码打包成jar。
5.  **注册发布:** 在打好jar包后，需要先添加资源，随后注册函数。注册成功后就可以在MaxCompute project explorer窗口的functions节点下查看该UDF，同时也可以在script editor里使用该UDF。

