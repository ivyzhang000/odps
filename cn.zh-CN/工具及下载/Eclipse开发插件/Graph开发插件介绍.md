# Graph开发插件介绍 {#concept_gcz_wcd_wdb .concept}

创建ODPS项目后，用户可以编写自己得Graph程序，依照以下步骤操作完成本地调试。 在此示例中，我们选用插件提供的“PageRank.java”来完成本地调试工作。 选中**examples**下的“PageRank.java”文件:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/3218_zh-CN.png)

右键单击，选择**Debug As** \> **ODPS MapReduce|Graph**:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/3220_zh-CN.png)

点击后出现对话框，作如下配置：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/3221_zh-CN.png)

查看作业运行结果：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/3222_zh-CN.png)

可以查看在本地的计算结果：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/3223_zh-CN.png)

调试通过后，用户可以将程序打包，并以Jar资源的形式上传到ODPS，并提交Graph作业。

**说明：** 

-   打包过程可参考[MapReduce Eclipse插件介绍](cn.zh-CN/工具及下载/Eclipse开发插件/MapReduce开发插件介绍.md)。
-   有关本地结果的目录结构介绍，也可以参考[MapReduce Eclipse插件介绍](cn.zh-CN/工具及下载/Eclipse开发插件/MapReduce开发插件介绍.md)。
-   关于上传Jar资源的详细介绍，可参考[基本操作](../cn.zh-CN/用户指南/常用命令/资源操作.md) 中创建Jar资源部分。
-   有关提交Graph作业，可参考 [Graph功能介绍](../cn.zh-CN/用户指南/图模型/功能概述.md)中Jar命令的介绍。

