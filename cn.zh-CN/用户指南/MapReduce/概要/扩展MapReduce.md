# 扩展MapReduce {#concept_ngt_sxf_vdb .concept}

传统的 MapReduce 模型要求每一轮 MapReduce 操作之后，数据必须落地到分布式文件系统上（比如 HDFS 或 MaxCompute 表）。而一般的 MapReduce 应用通常由多个 MapReduce 作业组成，每个作业结束之后需要写入磁盘，接下去的 Map 任务很多情况下只是读一遍数据，为后续的 Shuffle 阶段做准备，这样其实造成了冗余的 IO 操作。

MaxCompute 的计算调度逻辑可以支持更复杂编程模型， 针对上述的情况，可以在 Reduce 后直接执行下一次的 Reduce 操作，而不需要中间插入一个 Map 操作。因此，MaxCompute 提供了扩展的 MapReduce 模型，即可以支持 Map 后连接任意多个 Reduce 操作，比如 Map \> Reduce \> Reduce。

Hadoop Chain Mappper/Reducer 也支持类似的串行化 Map 或 Reduce 操作，但和 MaxCompute 的扩展 MapReduce（MR2）模型有本质的区别。

因为 Chain Mapper/Reducer 还是基于传统的 MapReduce 模型，只是可以在原有的 Mapper 或 Reducer 后面，再增加一个或多个 Mapper 操作（不允许增加Reducer）。这样的好处是：您可以复用之前的 Mapper 业务逻辑，可以把一个 Map 或 Reduce 拆成多个 Mapper 阶段，但本质上并没有改变底层的调度和 I/O 模型。

与 [MaxCompute](cn.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md) 相比，MR2 在 Map/Reduce 等函数编写方式上基本一致。较大的不同点发生在作业时。更多详情请参见 [扩展 MapReduce 示例](cn.zh-CN/用户指南/MapReduce/示例程序/Pipeline示例.md)。

