# 配置 MaxCompute Studio {#concept_nvh_s2y_5db .concept}

安装 MaxCompute Studio 插件后，可以在 IntelliJ IDEA 的 Setting 页面的左边栏找到 MaxCompute Studio 的配置项。打开 IntelliJ 配置页面的方法请参考 IntelliJ 的[文档](https://www.jetbrains.com/help/idea/2016.3/accessing-settings.html)。

## MaxCompute Studio 配置选项页 {#section_h45_v34_vdb .section}

MaxCompute Studio 配置选项页提供以下配置项：

1.  **本地元数据仓库存储路径**

    指定在本地存储 MaxCompute 项目空间元数据的路径。Studio 的缺省设置是本地用户目录下的.odps.studio/meta 这个隐含目录。

2.  **版本更新选项**

    -   复选框 **Automatically checks for new version** 可以控制 Studio 是否自动检查新版本更新
    -   按钮 **Check new versions** 用于手动检查新版本。 点击检查新版本按钮后，如果有新版本可以更新，将显示 **Install new version** 按钮，点击可以安装，安装完成后需要重启 IntelliJ IDEA。

## SDK & Console 配置选项页 {#section_k45_v34_vdb .section}

SDK & Console 配置选项页面提供以下配置项：

1.  **MaxCompute 客户端安装路径**

    指定本地安装的 MaxCompute 客户端的安装路径。 Studio 会检测路径中安装的 MaxCompute 客户端的版本，如果检测失败，会提示错误信息。

    **说明：** Studio 2.6.1 之后的版本自带了最新的 MaxCompute 客户端，用户不用特别指定。 如果用户希望使用自己特定版本的 MaxCompute客户端，可以指定路径。


## MaxCompute SQL 配置选项页 {#section_m45_v34_vdb .section}

MaxCompute SQL 配置选项页面提供以下配置项：

1.  **启动语法高亮**

    勾选**Enable syntax coloring** 选项，启动语法高亮功能。

2.  **启动代码自动补全**

    勾选**Enable code completion** 选项，启动代码自动补全功能。

3.  **启动代码格式化**

    勾选**Enable code formatting** 选项，启动代码格式化功能。

4.  **编译器选项**

    在这里这是全局缺省的编译器选项。 以下选项还可以在 SQL 编辑器的工具栏上为每个文件单独设置。

    -   **编译器模式（Compiler Mode）**
        -   单句模式（Statement Mode）：在该模式下，编译器对 SQL 文件单条语句作为单元进行编译、提交
        -   脚本模式（Script Mode）：在该模式下，编译器对整个 SQL 文件作为单元进行编译、提交。 *（注：脚本模式有利于编译器和优化器更大程度地优化执行计划，提高整体执行效率，目前处于测试阶段）*
    -   **类型系统**
        -   旧有类型系统（Legacy TypeSystem）：原有 MaxCompute 的类型系统
        -   MaxCompute 类型系统（MaxCompute TypeSystem）：MaxCompute 2.0 引入的新的类型系统
        -   Hive 类型系统（Hive Compatible TypeSystem）：MaxCompute 2.0 引入的 Hive 兼容模式下的类型系统
    -   **编译器版本**
        -   默认编译器（Default Version）：默认版本的编译器，
        -   实验性编译器（Flighting Version）：实验性的版本的编译器，包含正在测试中的编译器的新特性

## Account 配置选项页 {#section_s45_v34_vdb .section}

Account 配置选项页面提供添加和管理用户用于访问 MaxCompute 所用账户，参考 [用户认证](../DNODPS1898901/ZH-CN_TP_12094_V1.dita)：

Studio 需要通过用户指定的账号访问 MaxCompute 的项目空间和执行提交作业等操作，目前 Studio 支持的账号类型有：

-   阿里云账号（AccessKey）

**添加账户**

在 Account 配置选项页面，执行以下步骤：

1.  点击按钮 **+** 或者快捷键 `Ctrl-N`
2.  选择账户类型 **Aliyun Account by AccessKey**
3.  在弹出的 Add Account 窗口中填入:
    -   Account Name： 改账户在 Studio 中的标识名称。
    -   Using properties file: 从配置文件中读取 AccessKey ID 和 AccessKey Secret。
        -   选择一个在 [用户认证](../DNODPS1898901/ZH-CN_TP_12094_V1.dita) 中 `conf/odps_config.ini` 示例的配置文件。
    -   Using properties： 手工填入 AccessKey ID 和 AccessKey Secret。
        -   Access Id： 填入用户阿里云账号的 AccessKey ID。
        -   Access Key：填入用户阿里云账号的 AccessKey Secret。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12143/2461_zh-CN.png)

4.  点击按钮 **OK** 完成添加。 添加完成后账号会出现在 Account 配置选项页面的 Account 列表里。

**删除账户**

在 Account 配置选项页面，执行以下步骤（该操作仅在 Studio 配置中删除账户配置，对用户账户本身不产生影响）：

1.  在 Account 列表中选择要删除的账户名称。
2.  点击按钮 **-**。
3.  在弹出的的确认对话框中，选择**OK** 。

**修改账户 AccessKey**

在 Account 配置选项页面，执行以下步骤：

1.  在 Account 列表中选择要删除的账户名称。
2.  点击”铅笔”图标进行编辑。
3.  在弹出的 Edit Account 窗口中编辑 Account 信息，内容与上面的Add Account 窗口类似。

