# MaxCompute管家 {#concept_cqp_qcb_wdb .concept}

当开通MaxCompute预付费后，会遇到账号购买了150CU，但是经常看到使用预付费的项目很多任务依然要排队很长时间，管理员或运维人员希望能看到具体哪些任务抢占了资源，从而对任务进行合理的管控，如按任务对应的业务优先级进行调度时间调整，重要和次要任务错开调度的问题。

**MaxCompute 管家**就是解决这个预付费计算资源监控管理的问题。目前MaxCompute 管家主要提供三个功能，系统状态、资源组分配、任务监控。具体使用说明请参考DataWorks文档[MaxCompute预付费资源监控工具-CU管家](https://help.aliyun.com/document_detail/66564.html)。

**说明：** 

**MaxCompute管家使用须知：**

-   您购买了MaxCompute预付费CU资源，且购买数量为60CU或以上。

    **说明：** CU过小无法发挥计算资源及管家的优势。

-   MaxCompute管家目前仅支持华北2、华东2、华南1和华东1四个Region。

