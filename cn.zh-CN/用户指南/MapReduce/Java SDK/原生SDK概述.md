# 原生SDK概述 {#concept_dhw_rhl_vdb .concept}

本文将会为您介绍较为常用的 MapReduce 核心接口。

如果您使用 Maven，可以从 [Maven 库](http://search.maven.org/) 中搜索“odps-sdk-mapred”获取不同版本的 Java SDK，相关配置信息如下：

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-mapred</artifactId>
    <version>0.26.2-public</version>
</dependency>
```

|主要接口|描述|
|:---|:-|
|MapperBase|用户自定义的Map函数需要继承自此类。处理输入表的记录对 象，加工处理成键值对集合输出到Reduce阶段，或者不经过 Reduce阶段直接输出结果记录到结果表。不经过Reduce阶段而 直接输出计算结果的作业，也可称之为Map-Only作业。|
|ReducerBase|用户自定义的Reduce函数需要继承自此类。对与一个键（Key）关联的一组数值集（Values）进行归约计算。|
|TaskContext|是MapperBase及ReducerBase多个成员函数的输入参数之一。 含有任务运行的上下文信息。|
|JobClient|用于提交和管理作业，提交方式包括阻塞（同步）方式及非阻塞（异步） 方式。|
|RunningJob|作业运行时对象，用于跟踪运行中的MapReduce作业实例。|
|JobConf|描述一个MapReduce任务的配置，通常在主程序（main函数）中定义JobConf对象，然后通过JobClient提交作业给MaxCompute服务。|

## MapperBase {#section_dgt_z3l_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|void cleanup\(TaskContext context\)|在Map阶段结束时，map方法之后调用。|
|void map\(long key, Record record, TaskContext context\)|map方法，处理输入表的记录。|
|void setup\(TaskContext context\)|在Map阶段开始时，map方法之前调用。|

## ReducerBase {#section_jng_hjl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|void cleanup\( TaskContext context\)|在Reduce阶段结束时，reduce方法之后调用。|
|void reduce\(Record key, Iterator<Record \> values, TaskContext context\)|reduce方法，处理输入表的记录。|
|void setup\( TaskContext context\)|在Reduce阶段开始时，reduce方法之前调用。|

## TaskContext {#section_vch_kjl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|TableInfo\[\] getOutputTableInfo\(\)|获取输出的表信息|
|Record createOutputRecord\(\)|创建默认输出表的记录对象|
|Record createOutputRecord\(String label\)|创建给定label输出表的记录对象|
|Record createMapOutputKeyRecord\(\)|创建Map输出Key的记录对象|
|Record createMapOutputValueRecord\(\)|创建Map输出Value的记录对象|
|void write\(Record record\)|写记录到默认输出，用于Reduce端写出数据， 可以在Reduce端多次调用。|
|void write\(Record record, String label\)|写记录到给定标签输出，用于Reduce端写出数据。可以在 Reduce端多次调用。|
|void write\(Record key, Record value\)|Map写记录到中间结果，可以在Map函数中多次调用。 可以在Map端多次调用。|
|BufferedInputStream readResourceFileAsStream\(String resourceName\)|读取文件类型资源|
|Iterator<Record \> readResourceTable\(String resourceName\)|读取表类型资源|
|Counter getCounter\(Enum<? \> name\)|获取给定名称的Counter对象|
|Counter getCounter\(String group, String name\)|获取给定组名和名称的Counter对象|
|void progress\(\)|向MapReduce框架报告心跳信息。 如果用户方法处理时间 很长，且中间没有调用框架，可以调用这个方法避免task 超时，框架默认600秒超时。|

**说明：** MaxCompute 的 TaskContext 接口中提供了 progress 功能，但此功能是在防止 Worker 长时间运行未结束，被框架误认为超时而被杀的情况出现。此接口更类似于向框架发送心跳信息，并不是用来汇报 Worker 进度。MaxCompute MapReduce 默认 Worker 超时时间为 10 分钟（系统默认配置，不受用户控制），如果超过 10 分钟，Worker 仍然没有向框架发送心跳（调用 progress 接口），框架会强制停止该 Worker，MapReduce 任务失败退出。因此，建议您在 Mapper/Reducer 函数中，定期调用 progress 接口，防止框架认为 Worker 超时，误杀任务。

## JobConf {#section_nht_pjl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|void setResources\(String resourceNames\)|声明本作业使用的资源。只有声明的资源才能在运行Mapper/Reducer时通过TaskContext对象读取。|
|void setMapOutputKeySchema\(Column\[\] schema\)|设置Mapper输出到Reducer的Key属性|
|void setMapOutputValueSchema\(Column\[\] schema\)|设置Mapper输出到Reducer的Value属性|
|void setOutputKeySortColumns\(String\[\] cols\)|设置Mapper输出到Reducer的Key排序列|
|void setOutputGroupingColumns\(String\[\] cols\)|设置Key分组列|
|void setMapperClass\(Class<? extends Mapper \> theClass\)|设置作业的Mapper函数|
|void setPartitionColumns\(String\[\] cols\)|设置作业指定的分区列. 默认是Mapper输出Key的所有列|
|void setReducerClass\(Class<? extends Reducer \> theClass\)|设置作业的Reducer|
|void setCombinerClass\(Class<? extends Reducer \> theClass\)|设置作业的combiner。在Map端运行，作用类似于单个Map 对本地的相同Key值做Reduce|
|void setSplitSize\(long size\)|设置输入分片大小，单位 MB，默认256|
|void setNumReduceTasks\(int n\)|设置Reducer任务数，默认为Mapper任务数的1/4|
|void setMemoryForMapTask\(int mem\)|设置Mapper任务中单个Worker的内存大小，单位：MB， 默认值2048。|
|void setMemoryForReduceTask\(int mem\)|设置Reducer任务中单个Worker的内存大小，单位：MB， 默认值 2048。|

**说明：** 

-   通常情况下，GroupingColumns 包含在 KeySortColumns 中，KeySortColumns 和 PartitionColumns 要包含在 Key 中。
-   在 Map 端，Mapper 输出的 Record 会根据设置的 PartitionColumns 计算哈希值，决定分配到哪个 Reducer，会根据 KeySortColumns 对 Record 进行排序。
-   在 Reduce 端，输入 Records 在按照 KeySortColumns 排序好后，会根据 GroupingColumns 指定的列对输入的 Records 进行分组，即会顺序遍历输入的 Records，把 GroupingColumns 所指定列相同的 Records 作为一次 reduce 函数调用的输入。

## JobClient {#section_jz2_5jl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|static RunningJob runJob\(JobConf job\)|阻塞（同步）方式提交MapReduce作业后立即返回|
|static RunningJob submitJob\(JobConf job\)|非阻塞（异步）方式提交MapReduce作业后立即返回|

## RunningJob {#section_wnp_vjl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|String getInstanceID\(\)|获取作业运行实例ID，用于查看运行日志和作业管理。|
|boolean isComplete\(\)|查询作业是否结束。|
|boolean isSuccessful\(\)|查询作业实例是否运行成功。|
|void waitForCompletion\(\)|等待直至作业实例结束。一般使用于异步方式提交的作业。|
|JobStatus getJobStatus\(\)|查询作业实例运行状态。|
|void killJob\(\)|结束此作业。|
|Counters getCounters\(\)|获取Conter信息。|

## InputUtils {#section_udl_xjl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|static void addTable\(TableInfo table, JobConf conf\)|添加表table到任务输入，可以被调用多次 ，新加入的表以append方式添加到输入队列中。|
|static void setTables\(TableInfo \[\] tables, JobConf conf\)|添加多张表到任务输入中。|

## OutputUtils {#section_uzf_zjl_vdb .section}

**主要函数接口**：

|主要接口|描述|
|:---|:-|
|static void addTable\(TableInfo table, JobConf conf\)|添加表table到任务输出，可以被调用多次 ，新加入的表以append方式添加到输出队 列中。|
|static void setTables\(TableInfo \[\] tables, JobConf conf\)|添加多张表到任务输出中。|

## Pipeline {#section_zld_bkl_vdb .section}

Pipeline是 [MR2](cn.zh-CN/用户指南/MapReduce/概要/扩展MapReduce.md) 的主体类。可以通过 Pipeline.builder 构建一个 Pipeline。Pipeline 的主要接口如下：

```
    public Builder addMapper(Class<? extends Mapper> mapper)
    public Builder addMapper(Class<? extends Mapper> mapper,
           Column[] keySchema, Column[] valueSchema, String[] sortCols,
           SortOrder[] order, String[] partCols,
           Class<? extends Partitioner> theClass, String[] groupCols)
    public Builder addReducer(Class<? extends Reducer> reducer)
    public Builder addReducer(Class<? extends Reducer> reducer,
           Column[] keySchema, Column[] valueSchema, String[] sortCols,
           SortOrder[] order, String[] partCols,
           Class<? extends Partitioner> theClass, String[] groupCols)
    public Builder setOutputKeySchema(Column[] keySchema)
    public Builder setOutputValueSchema(Column[] valueSchema)
    public Builder setOutputKeySortColumns(String[] sortCols)
    public Builder setOutputKeySortOrder(SortOrder[] order)
    public Builder setPartitionColumns(String[] partCols)
    public Builder setPartitionerClass(Class<? extends Partitioner> theClass)
    public Builder setOutputGroupingColumns(String[] cols)
```

示例如下：

```
    Job job = new Job();
    Pipeline pipeline = Pipeline.builder()
     .addMapper(TokenizerMapper.class)
     .setOutputKeySchema(
         new Column[] { new Column("word", OdpsType.STRING) })
     .setOutputValueSchema(
         new Column[] { new Column("count", OdpsType.BIGINT) })
     .addReducer(SumReducer.class)
     .setOutputKeySchema(
         new Column[] { new Column("count", OdpsType.BIGINT) })
     .setOutputValueSchema(
         new Column[] { new Column("word", OdpsType.STRING),
         new Column("count", OdpsType.BIGINT) })
     .addReducer(IdentityReducer.class).createPipeline();
    job.setPipeline(pipeline);  
    job.addInput(...)
    job.addOutput(...)
    job.submit();
```

如上所示，您可以在 main 函数中构建一个 Map 后连续接两个 Reduce 的 MapReduce 任务。如果您比较熟悉 MapReduce 的基础功能，即可轻松使用 MR2。

**说明：** 

-   建议您在使用 MR2 功能前，首先了解 MapReduce 的基础用法。
-   JobConf 仅能够配置 Map 后接单 Reduce 的 MapReduce 任务。

## 数据类型 {#section_bqj_3kl_vdb .section}

MapReduce 支持的数据类型有：Bigint，String，Double，Boolean，datetime 和 Decimal 类型。MaxCompute 数据类型与 Java 数据类型的对应关系，如下所示：

|MaxCompute SQL Type|Bigint|String|Double|Boolean|Datetime|Decimal|
|:------------------|:-----|:-----|:-----|:------|:-------|:------|
|Java Type|Long|String|Double|Boolean|Date|BigDecimal|

