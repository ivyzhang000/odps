# 如何通过 Studio 管理 MaxCompute 元数据 {#concept_p3g_jm5_vdb .concept}

MaxCompute用户在日常使用过程中，需要浏览管理所在project的meta（包括表，函数和资源），我们来看看如何通过Studio来方便的管理操作。

-   **如何新增一个project连接:** 参考[新增连接](cn.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md) 通过MaxCompute project explorer新建一个MaxCompute project连接，连接连通性测试成功后，就可以看到刚才新增的project节点树。
-   **如何查看table列表及schema:** 参考[浏览meta](cn.zh-CN/工具及下载/MaxCompute Studio/管理数据和资源/浏览表及 UDF.md) 展开project下的tables & views能看到table和view列表，展开具体的某个table能看到列和类型。
-   **如何查看function代码:** 展开functions能看到function列表，展开具体的某个function能看到方法签名，双击该function即可以看到反编译的源码。
-   **如何查看下载表的sample数据:** 参考[导入导出表数据](cn.zh-CN/工具及下载/MaxCompute Studio/管理数据和资源/导入导出数据.md) 双击某个table就能看到详细的schema，还能预览sample数据，在data preview窗格中右键**Export grid data**。你也可以直接在节点树的表名上右键**export**。
-   **如何更新节点的meta:** 譬如某个表新增了一列，你可以在选中该表后右键**refresh meta**或点击工具栏上的**refresh**按钮，即可以看到新增的这一列了。又譬如project下新建了一些表，选中**tables & view**后点击工具栏上的**refresh**按钮即可以看到新建的这些表了。
-   **如何对表做DDL操作:** 删除是可以直接批量操作的，选中表后，右键**Delete table from server**即可。新增和编辑还不支持直接通过界面操作，你可以通过在editor里编写相应的DDL语句来完成。

