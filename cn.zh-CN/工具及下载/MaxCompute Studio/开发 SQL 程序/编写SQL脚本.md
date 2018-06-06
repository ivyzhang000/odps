# 编写SQL脚本 {#concept_nvh_s3d_5db .concept}

[MaxCompute Studio 模块创建完成后](ZH-CN_TP_12125_V1.dita)，即可开始编写 MaxCompute SQL 脚本。

## 操作步骤 {#section_vpm_rlf_vdb .section}

1.  右击**scripts** ，导航至**New** \> **MaxCompute Script** 。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1845_zh-CN.png)

2.  填写弹出框中的相关内容，点击**OK**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1846_zh-CN.png)

    -   Script Name：脚本名称。
    -   Script type：脚本类型。
    -   Target Project：目标 MaxCompute 项目。单击后面的 **+** 即可新建一个MaxCompute Project，配置详情请参见 [新建项目空间连接](ZH-CN_TP_12119_V1.dita)。
3.  在 SQL 文件编辑界面中，编写 SQL。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1849_zh-CN.png)

    **说明：** 

    1.  实际 SQL 请根据自己的 MaxCompute Project 中的表进行编写。可单击 toolbar 右上角切换绑定的不同的 MaxCompute 项目，也支持跨 project 资源依赖。例如 script 绑定了 ProjectA，同时还会用到 ProjectB.table1，这时 Studio 会自动使用 ProjectA 的账号去抓取 ProjectB 的元数据。表的元数据 Studio 会保存在本机中类似下图的位置：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1850_zh-CN.png)

    2.  新建sql script时的代码模板是可以在Intellij Preferences页修改的：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1852_zh-CN.png)


## MaxCompute Studio 功能 {#section_czx_y4f_vdb .section}

MaxCompute Studio 不仅提供语法高亮，智能提醒，错误提示，还支持以下功能：

-   **schema annotator**： 当鼠标悬停在表上，可显示其schema；悬停在列上，可显示其类型；悬停在函数上，可显示其签名。
-   **code folding**： 可以将子查询等折叠起来，方便长 SQL 的阅读。
-   **brace matching**：鼠标单击高亮左括号，其匹配的右括号也会高亮，反之亦然。
-   **go to declaration**：按住 Ctrl 键，单击**table** ，即可查看 table 详情。单击**function** ，即可显示其源码。
-   **code formatting**：支持对当前脚本格式化，快捷键（Ctrl + Alt + L）。可在如下页面自定义格式化规则，譬如关键字大小写，是否换行等。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1853_zh-CN.png)

-   **code inspect**：支持对当前脚本进行代码检查，某些检查还会提供quick fix，可通过快捷键Alt + Enter唤出。另外，可在Preference - Editor - Inspections - MaxCompute 处修改某条规则。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1854_zh-CN.png)

-   **find usages**：选中 editor 中的某张表（或函数），右键菜单选**Find Usages** ，则会在当前 IntelliJ project 下寻找所有使用该表的脚本。
-   **live template**：Studio 内置了一些 SQL live template，可以在编辑器中使用 Ctrl + J \(Command + J on Mac OS X\) 快捷键唤出（例如忘记了 insert into table 的语法，便可唤出 live template popup 后搜索 insert table）。
-   **builtin documentation**：支持在系统内置函数处通过 Ctrl + Q （Ctrl + J on Mac OS X）唤出帮助文档。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1856_zh-CN.png)

-   **sql history**：所有通过studio提交运行的sql我们都记录在本地了，你可以点击toolbar上的图标，弹出sql history窗口，查询你曾经执行过的SQL。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12126/1857_zh-CN.png)


