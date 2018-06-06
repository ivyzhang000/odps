# 提交 SQL 脚本 {#concept_iwn_lsf_vdb .concept}

MaxCompute Studio 可直接将 MaxCompute SQL 提交到服务端运行，并显示查询结果，执行计划等详细信息。它在提交前会进行编译，能够有效避免提交到服务端后才发现编译错误。

## 前提条件 {#section_zrx_zsf_vdb .section}

-   首先 [创建 MaxCompute 项目连接](ZH-CN_TP_12119_V1.dita)，并绑定目标项目。

-   创建[MaxCompute Studio Module](ZH-CN_TP_12125_V1.dita)。

-   在提交前需根据自身需求进行相关设置。MaxCompute Studio 提供了丰富的设置功能，可在 Editor 编辑页面上方的 Tool Bar 工具栏中快速设置。设置主要分为以下三种：

    -   **编辑器模式**：编译器模式设定包括两种模式，脚本模式和单步模式。

        -   单步模式会将提交的脚本文件按`；`分隔，逐条提交到服务端执行。

        -   脚本模式为最新开发模式，可将整条脚本一次提交到服务端，由服务端提供整体优化，效率更高，推荐使用。

    -   **类型系统**：类型系统主要解决 SQL 语句的兼容性问题，分为以下三种类型：

        -   旧有类型系统：原有 MaxCompute 的类型系统。

        -   MaxCompute 类型系统：MaxCompute 2.0 引入的新的类型系统。

        -   Hive 类型系统：MaxCompute 2.0 引入的 Hive 兼容模式下的类型系统。

    -   **编译器版本**：MaxCompute Studio 提供稳定版编译器和实验性编译器两种模式：

        -   默认编译器：稳定版本。

        -   实验性编译器：包含编译器最新特性。

        **说明：** 脚本提交设置可采用 **全局设定**，导航至**File** \> **Settings** \> **MaxCompute** ，选择 **MaxCompute SQL**，其中 **Compiler** \> **Submit** 中可以设置以上属性。


## 提交SQL脚本 {#section_pyn_r5f_vdb .section}

Editor 上方工具栏中提供同步，编译，提交功能。

```
* **同步功能**：更新 SQL 脚本中使用的元数据，包括表名、UDF 等。如果 Studio 提示表或函数找不到，而服务端又明确存在时，可尝试使用该功能更新元数据。
* **编译、提交**：按 MaxCompute SQL 预发规则编译或提交到服务端，编译错误会在 MaxCompute Compiler 窗口中显示详细信息。
```

**具体操作**

1.  编写好 SQL 语句后，单击工具栏或侧边栏上的绿色运行图标，或者右键 **Script Editor**，选择**Run MaxCompute SQL Script** ，即可提交到服务端。当 SQL 中存在变量时（如下图的$\{bizdate\}），会弹出对话框，提示您输入变量值。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12127/1916_zh-CN.png)

2.  SQL会先被本地编译\(依赖于你在**Project Explorer**窗口中添加的项目元数据\)，无编译错误后就会提交到服务端执行。sql执行过程中会显示运行日志，当已经开始在服务端运行时，会打开job detail页，显示作业运行的基本信息以及执行图：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12127/1917_zh-CN.png)

3.  可以在结果页查看SQL结果，单句模式下当存在多条语句时，会显示每条语句的结果。可以选择表格中的一些行或列，Ctrl + C到剪切板中。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12127/1920_zh-CN.png)


