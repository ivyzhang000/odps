# MapReduce概述 {#concept_fml_smf_vdb .concept}

MaxCompute 提供了三个版本的 MapReduce 编程接口，如下所示：

-   MaxCompute MapReduce：MaxCompute 的原生接口，执行速度更快。开发更便捷，不暴露文件系统。

-   [MR2](cn.zh-CN/用户指南/MapReduce/概要/扩展MapReduce.md)（扩展 MapReduce）：对 MaxCompute MapReduce 的扩展，支持更复杂的作业调度逻辑。Map/Reduce 的实现方式与 MaxCompute 原生接口一致。

-   以及 [Hadoop 兼容版本](cn.zh-CN/用户指南/MapReduce/概要/开源兼容MapReduce.md)：高度兼容 [Hadoop MapReduce](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html) ，与 MaxCompute 原生 MapReduce，[MR2](https://help.aliyun.com/document_detail/27876.html) 不兼容。


以上三个版本在 [基本概念](cn.zh-CN/用户指南/MapReduce/功能介绍/基本概念.md)，[作业提交](cn.zh-CN/用户指南/MapReduce/功能介绍/作业提交.md)，[输入输出](cn.zh-CN/用户指南/MapReduce/功能介绍/输入与输出.md)，[资源使用](cn.zh-CN/用户指南/MapReduce/功能介绍/资源使用.md) 等方面基本一致，不同的是 Java SDK 彼此不同。本文仅对 MapReduce 的基本原理做简单介绍，更多详情请参见 [Hadoop MapReduce 教程](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html)。

**说明：** 您还不能够通过 MapReduce 读写“外部表”中的数据。

## 应用场景 {#section_m5s_rwf_vdb .section}

MapReduce 最早是由 Google 提出的分布式数据处理模型，随后受到了业内的广泛关注，并被大量应用到各种商业场景中。示例如下：

-   搜索：网页爬取、倒排索引、PageRank。
-   Web 访问日志分析：
    -   分析和挖掘用户在 Web 上的访问、购物行为特征，实现个性化推荐。
    -   分析用户访问行为。
-   文本统计分析：
    -   莫言小说的 WordCount、词频 TFIDF 分析。
    -   学术论文、专利文献的引用分析和统计。
    -   维基百科数据分析等。
-   海量数据挖掘：非结构化数据、时空数据、图像数据的挖掘。
-   机器学习：监督学习、无监督学习、分类算法如决策树、SVM 等。
-   自然语言处理：
    -   基于大数据的训练和预测。
    -   基于语料库构建单词同现矩阵，频繁项集数据挖掘、重复文档检测等。
-   广告推荐：用户点击（CTR）和购买行为（CVR）预测。

## 处理流程 {#section_l5j_vwf_vdb .section}

MapReduce 处理数据过程主要分成 Map 和 Reduce 两个阶段。首先执行 Map 阶段，再执行 Reduce 阶段。Map 和 Reduce 的处理逻辑由用户自定义实现，但要符合 MapReduce 框架的约定。处理流程如下所示：

1.  在正式执行 Map 前，需要将输入数据进行 **分片**。所谓分片，就是将输入数据切分为大小相等的数据块，每一块作为单个 Map Worker 的输入被处理，以便于多个 Map Worker 同时工作。
2.  分片完毕后，多个 Map Worker 便可同时工作。每个 Map Worker 在读入各自的数据后，进行计算处理，最终输出给 Reduce。Map Worker 在输出数据时，需要为每一条输出数据指定一个 Key，这个 Key 值决定了这条数据将会被发送给哪一个 Reduce Worker。Key 值和 Reduce Worker 是多对一的关系，具有相同 Key 的数据会被发送给同一个 Reduce Worker，单个 Reduce Worker 有可能会接收到多个 Key 值的数据。
3.  在进入 Reduce 阶段之前，MapReduce 框架会对数据按照 Key 值排序，使得具有相同 Key 的数据彼此相邻。如果您指定了 **合并操作（Combiner）**，框架会调用 Combiner，将具有相同 Key 的数据进行聚合。Combiner 的逻辑可以由您自定义实现。与经典的 MapReduce 框架协议不同，在 MaxCompute 中，Combiner 的输入、输出的参数必须与 Reduce 保持一致，这部分的处理通常也叫做 **洗牌（Shuffle）**。
4.  接下来进入 Reduce 阶段。相同 Key 的数据会到达同一个 Reduce Worker。同一个 Reduce Worker 会接收来自多个 Map Worker 的数据。每个 Reduce Worker 会对 Key 相同的多个数据进行 Reduce 操作。最后，一个 Key 的多条数据经过 Reduce 的作用后，将变成一个值。

**说明：** 上文仅是对 MapReduce 框架做简单介绍，更多详情请查阅其他相关资料。

下文将以 WordCount 为例，为您介绍 MaxCompute MapReduce 各个阶段的概念。

假设存在一个文本 a.txt，文本内每行是一个数字，您要统计每个数字出现的次数。文本内的数字称为 Word，数字出现的次数称为 Count。如果 MaxCompute Mapreduce 完成这一功能，需要经历以下流程，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12013/1922_zh-CN.jpg)

操作步骤：

1.  输入数据：对文本进行分片，将每片内的数据作为单个 Map Worker 的输入。
2.  Map 阶段：Map 处理输入，每获取一个数字，将数字的 Count 设置为 1，并将此<Word, Count\>对输出，此时以 Word 作为输出数据的 Key。
3.  Shuffle \> 合并排序：在 Shuffle 阶段前期，首先对每个 Map Worker 的输出，按照 Key 值（即 Word 值）进行排序。排序后进行 Combiner 操作，即将 Key 值（Word 值）相同的 Count 累加，构成一个新的<Word, Count\>对。此过程被称为合并排序。
4.  Shuffle \> 分配 Reduce：在 Shuffle 阶段后期，数据被发送到 Reduce 端。Reduce Worker 收到数据后依赖 Key 值再次对数据排序。
5.  Reduce 阶段：每个 Reduce Worker 对数据进行处理时，采用与 Combiner 相同的逻辑，将 Key 值（Word 值）相同的 Count 累加，得到输出结果。
6.  输出结果数据。

**说明：** 由于 MaxCompute 的所有数据都被存放在表中，因此 MaxCompute MapReduce 的输入、输出只能是表，不允许您自定义输出格式，不提供类似文件系统的接口。

