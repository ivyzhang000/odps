# SDK概述 {#concept_ygm_k4m_vdb .concept}

如果您使用 Maven，可以从 [Maven 库](http://search.maven.org/) 中搜索“odps-sdk-graph”来获取不同版本的 Java SDK。相关配置信息，如下所示：

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-graph</artifactId>
    <version>0.20.7</version>
</dependency>
```

|主要接口|说明|
|:---|:-|
|GraphJob|GraphJob继承自JobConf，用于定义、提交和管理一个 MaxCompute Graph 作业。|
|Vertex|Vertex 是图的点的抽象，包含属性：id，value，halted，edges，通过 GraphJob 的 setVertexClass 接口提供 Vertex 实现。|
|Edge|Edge 是图的边的抽象，包含属性：destVertexId, value，图数据结构采用邻接表，点的出边保存在点的 edges 中。|
|GraphLoader|GraphLoader 用于载入图，通过 GraphJob 的 setGraphLoaderClass 接口提供 GraphLoader 实现。|
|VertexResolver|VertexResolver 用于自定义图拓扑修改时的冲突处理逻辑，通过GraphJob的 setLoadingVertexResolverClass 和 setComputingVertexResolverClass 接口提供图加载和迭代计算过程中的图拓扑修改的冲突处理逻辑。|
|Partitioner|Partitioner 用于对图进行划分使得计算可以分片进行，通过GraphJob的 setPartitionerClass 接口提供 Partitioner 实现，默认采用 HashPartitioner，即对点 ID 求哈希值然后对 Worker 数目取模。|
|WorkerComputer|WorkerComputer允许在 Worker 开始和退出时执行用户自定义的逻辑，通过GraphJob的 setWorkerComputerClass 接口提供WorkerComputer 实现。|
|Aggregator|Aggregator 的 setAggregatorClass\(Class …\) 定义一个或多个 Aggregator。|
|Combiner|Combiner 的 setCombinerClass 设置 Combiner。|
|Counters|计数器，在作业运行逻辑中，可以通过 WorkerContext 接口取得计数器并进行计数，框架会自动进行汇总。|
|WorkerContext|上下文对象，封装了框架的提供的功能，如修改图拓扑结构，发送消息，写结果，读取资源等。|

