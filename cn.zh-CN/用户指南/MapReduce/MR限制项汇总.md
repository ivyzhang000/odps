# MR限制项汇总 {#concept_m4c_4lg_vdb .concept}

为避免您因没注意限制条件，业务启动后才发现限制条件，导致业务停止的现象发生，本文将对 MaxCompute MR 限制项进行汇总，以方便您查看。

MaxCompute MR 限制项汇总，如下表所示：

|边界名|边界值|分类|配置项名称|默认值|是否可配置|说明|
|:--|:--|:-|:----|:--|:----|:-|
|instance 内存占用|\[256MB，12GB\]|内存限制|odps.stage.mapper\(reducer\).mem 和 odps.stage.mapper\(reducer\).jvm.mem|2048M＋1024M|是|单个 map instance 或 reduce instance 占用 memory，由框架 memory（默认2048M）和 jvm的heap memory（默认 1024M）两部分 。|
|resource 数量|256个|数量限制|不涉及|无|否|单个 job 引用的 resource 数量不超过 256 个，table、archive 按照一个单位计算 。|
|输入路数和输出路数|1024 个和 256 个|数量限制|不涉及|无|否|单个job 的输入路数不能超过 1024（同一个表的一个分区算一路输入，总的不同表个数不能超过 64 个），单个 job 的输出路数不能超过 。|
|counter 数量|64个|数量限制|不涉及|无|否|单个 job 中自定义 counter 的数量不能超过 64，counter 的 group name 和 counter name 中不能带有 \#，两者长度和不能超过 100 。|
|map instance 　|\[1，100000\]|数量限制|odps.stage.mapper.num|无|是|单个 job 的 map instance 个数由框架根据 split size 计算得出，如果没有输入表，可以通过 odps.stage.mapper.num 直接设置，最终个数范围 \[1, 100000\].|
|reduce instance|\[0，2000\]|数量限制|odps.stage.reducer.num|无|是|单个 job 默认 reduce instance 个数为 map instance 个数的1/4，用户设置作为最终的 reduce instance 个数，范围 \[0, 2000\]。可能出现这样的情形：reduce 处理的数据量会比 map 大很多倍，导致 reduce 阶段比较慢，而 reduce 只能最多起 2000 。|
|重试次数|3|数量限制|不涉及|无|否|单个 map instance 或reduce instance 失败重试次数为3，一些不可重试的异常会直接导致作业失败。|
|local debug模式|instance个数不超100|数量限制|不涉及|无|否|local debug 模式下，默认 map instance 个数为 2，不能超过 100；默认reduce instance 个数为 1，不能超过 100；默认一路输入下载记录数 100，不能超过 10000 。|
|重复读取resource次数|64次|数量限制|不涉及|无|否|单个 map instance 或 reduce instance 重复读一个 resource 次数限制<=64 次 。|
|resource 字节数|2G|长度限制|不涉及|无|否|单个 job 引用的resource 总计字节数大小不超过 2G。|
|split size|\[1，\)|长度限制|odps.stage.mapper.split.size|256M|是|框架会参考设置的 split size 值来划分 map，决定 map 的个数 。|
|string 列内容长度|8M|长度限制|不涉及|无|否|MaxCompute 表string 列内容长度不允许 。|
|worker 运行超时时间|［1，3600］|时间限制|odps.function.timeout|600|是|map 或者 reduce worker 在无数据读写且没有通过 context.progress\(\)主动发送心态的情况下的超时时间，默认值是 600s。|
|MR引用Table资源支持的字段类型|BIGINT、DOUBLE、STRING、DATETIME、BOOLEAN|数据类型限制|不涉及|无|否|MR任务引用到Table资源时，若table表字段有其他类型字段执行报错|

