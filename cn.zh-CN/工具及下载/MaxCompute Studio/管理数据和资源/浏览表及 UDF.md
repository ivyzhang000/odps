# 浏览表及 UDF {#concept_cbj_cmz_5db .concept}

## 查看表及函数 {#section_wf5_mmz_5db .section}

在 **项目空间浏览器（Project Explorer\)** 窗口中可以快速浏览已添加连接的表、函数、资源等。使用前提是 [添加 MaxCompute 项目连接](cn.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md)。

## 浏览表和函数 {#section_xf5_mmz_5db .section}

要浏览项目空间的表和函数，使用以下步骤：

1.  打开 **项目空间浏览器（Project Explorer\)** ，即可以查看已添加的 Project 节点树。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1621_zh-CN.png)

    节点树上方是工具栏，包括：

    -   **增加Project**：新增一个到 MaxCompute 项目空间的连接
    -   **删除Project**：删除一个项目空间浏览器中的项目连接，对服务器端项目空间无影响
    -   **刷新元数据**：从服务器端项目空间刷新元数据信息，刷新本地元数据缓存
    -   **展开节点**：展开全部树节点
    -   **折叠节点**：折叠全部树节点
    -   **用户反馈**：提交用户反馈
    -   **在线文档**：打开在线文档
2.  双击或点击下拉箭头展开 **Tables** 节点，可列出该项目下的所有表（包括虚拟视图）。这里的表名列表与用户执行show tables命令相等，需要用户在project下有list table权限。函数（**Functions**）和资源（**Resources**）节点类似:

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1623_zh-CN.png)

3.  Studio 会将服务上的项目元数据下载到本地，当服务端元数据有更新时，如新增了一张表，需手动触发一次刷新，将变化的元数据重新加载到本地。 可以选择在项目（Project）或表（Table）级别做刷新，步骤如下：

    1.  选中相应的节点。

    2.  点击工具栏上的刷新图标或在右键菜单中选择刷新菜单项。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1624_zh-CN.png)


## 查看表详细信息 {#section_dg5_mmz_5db .section}

用户可以通过 Studio 的 表详情视图（Table Details View） 查看数据表详细信息。

1.  在节点树中，展开某个表名节点，可快速查看列名和类型：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1625_zh-CN.png)

2.  双击某个表或右键菜单**Show Table Detail** 可以查看表的详细信息，包括 owner，size，column 等元数据；表结构信息； 以及 data preview：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1626_zh-CN.png)

3.  通过在**Tables & Views**右键菜单项 **Open specific entity** ，可以指定表名显示详情（注意要完整表名称）。另外如果用户没有project的list权限，而只有具体某张表的权限，也可以通过这种方式将该表抓取下来。函数（Functions）及资源（Resources）类似。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1627_zh-CN.png)

    Intellij IDE 默认支持搜索，可展开表后直接敲击键盘模糊搜索。

4.  studio也支持快速搜索某张表，可通过快捷键\(win:Ctrl+Alt+Shift+N mac:⌥+⌘+O\)唤出navigate symbol，输入表名后回车即可。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1628_zh-CN.png)

    可通过前置关键字\(table:或function:或resource:\)缩小搜索范围，譬如想搜索函数count，可输入function:count。

5.  当想知道某张表在哪些script中用到时，可以右键该表，使用Find Usages功能。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1629_zh-CN.png)


## 查看函数详细信息 {#section_wss_3pz_5db .section}

1.  在 **Functions** 树UserDefined节点下可以展开某个函数节点，以显示该函数的方法签名。双击某个函数节点（或在 **Resources** 下双击改函数对应的源码资源），可打开该函数对应的的代码。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1630_zh-CN.png)

    **说明：** Java代码通过反编译jar获取，并非源码。Python UDF解析签名需要安装pyodps\(MaxCompute python sdk\)，具体的先安装pip: sudo /usr/bin/python get-pip.py \(请自行google下载get-pip.py\)，然后安装pyodps: sudo /usr/bin/python -m pip install pyodps。

2.  在**Functions** 树BuiltIn节点下分类显示了系统内置函数，展开显示签名，双击显示函数文档。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12121/1631_zh-CN.png)


