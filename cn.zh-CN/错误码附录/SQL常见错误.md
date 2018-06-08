# SQL常见错误 {#concept_c4b_42w_tdb .concept}

|错误信息|触发条件|
|:---|:---|
|ODPS-0110005:Unknown exception from metadata operation|在顶层捕获所有未知异常|
|ODPS-0110011:Authorization exception|权限不足|
|ODPS-0110021:Invalid parameters|参数有误|
|ODPS-0110031:Invalid object type|非法对象类型|
|ODPS-0110041:Invalid meta operation - AlreadyExistsException\(message:Partition already exists, existed values|目前MaxCompute对操作的表没有加锁，这个错误是由Meta产生竞争导致，即向同一个分区同时多次操作读写操作容易产生此类错误。因此建议用户在MaxCompute还没有锁机制的情况下，先不要同时对一个表做操作。|
|ODPS-0110041:Invalid meta operation - No value returned from meta UDF|无效meta操作|
|ODPS-0110061: Failed to run ddltask - AlreadyExistsException\(message:Partition already exists, existed values:\)|目前MaxCompute对操作的表没有加锁，这个错误是由Meta产生竞争导致，即向同一个分区同时多次操作读写操作容易产生此类错误。因此建议用户在MaxCompute还没有锁机制的情况下，先不要同时对一个表做操作。|
|ODPS-0110061: Failed to run ddltask - SimpleLock conflict failure, add partition is already on-going|当用户批量添加统一分区时，会出现此错误。MaxCompute仅会执行接收到的第一个添加分区命令，并忽略后续请求。|
|ODPS-0110071：OTS initialization exception|OTS初始化异常|
|ODPS-0110081：OTS transaction exception|OTS TRANSACTION异常|
|ODPS-0110091：OTS filtering exception|OTS 条件筛选异常|
|ODPS-01100101：OTS processing exception|OTS执行异常|
|ODPS-01100111：OTS invalid data object|OTS错误数据|
|ODPS-01100121： Unknown OTS exception|OTS未知错误|
|ODPS-0110131:StorageDescriptor compression exception|StorageDescriptor 压缩异常|
|ODPS-0110141:Data version exception|数据版本异常|
|ODPS-0110999:Critical! Internal error happened in commit operation and rollback failed, possible breach of atomicity|无法回滚|
|ODPS-0120005:Unknown exception from processor|在顶层捕获所有未知异常|
|ODPS-0120011:Authorization exception|权限不足|
|ODPS-0120021:the delimitor must be the same in wm\_concat|相同group中separator不同|
|ODPS-0120031：Instance has been cancelled|某instance被cancel多次|
|ODPS-0121011:Invalid regular expression pattern|[内建函数](../cn.zh-CN/用户指南/SQL/内建函数/数学函数.md)中的正则处理函数接收到了不能识别的正则表达式。|
|ODPS-0121021:Regexec call failed|匹配正则时引起的错误|
|ODPS-0120031：Instance has been cancelled|某instance被cancel多次|
|ODPS-0121035:Illegal implicit type cast|类型转换错误。一般为不支持的隐式类型转换错误。通常情况下是由于违背[隐式类型转换](../cn.zh-CN/用户指南/SQL/SQL概述.md)规则引起的。[内建函数](../cn.zh-CN/用户指南/SQL/内建函数/数学函数.md)也有不同的隐式转换规则。|
|ODPS-0121045:Unsupported return type|不支持的返回值|
|ODPS-0121055:Empty argument value|参数是空串或者NULL|
|ODPS-0121065:Argument value out of range|参数值错误|
|ODPS-0121075:Invalid number of arguments|参数个数不合法|
|ODPS-0121081:Illegal argument type|参数基本类型错误|
|ODPS-0121095:Invalid arguments|其他参数错误|
|ODPS-0121105:Constant argument value expected|需要输入常数，但输入列名|
|ODPS-0121115:Column reference expected|需要输入列名，但输入常数|
|ODPS-0121125:Unsupported function or operation|不支持的UDF或其它操作|
|ODPS-0121135:Malloc memory failed|内存分配异常|
|ODPS-0121145:Data overflow|数据溢出，超出数据类型的[值域范围](../cn.zh-CN/产品简介/基本概念/数据类型.md)。通常情况下有可能是”除零”错误或聚合函数，例如：sum导致的数据移出。|
|ODPS-0123019:Distributed file operation exception|磁盘读写异常|
|ODPS-0123023:Unsupported reduce type|不支持的Reduce|
|ODPS-0123031:Partition exception|分区异常|
|ODPS-0123043:buffer overflow|缓存溢出|
|ODPS-0123055:Script exception|脚本异常|
|ODPS-0123065:Join exception|Join异常|
|ODPS-0123075:Hash exception|哈希异常|
|ODPS-0123081:Invalid datetime string|datetime 字符串异常|
|ODPS-0123091:Illegal type cast|非法类型转换。通常情况下，是由于非法的[显式类型转换](../cn.zh-CN/用户指南/SQL/SQL概述.md)造成的。|
|ODPS-0123105： Job got killed|作业被kill|
|ODPS-0123111:Format string does not match datetime string|格式串不匹配日期字符串。用户在SQL中手动输入的日期格式不符合MaxCompute的格式要求，或者对DataTime相关内建函数使用不当。MaxCompute DataTime类型的显示格式可参考[数据类型](../cn.zh-CN/产品简介/基本概念/数据类型.md)中的示例，或[内建函数](../cn.zh-CN/用户指南/SQL/内建函数/数学函数.md)中有关日期函数的介绍。|
|ODPS-0123121:Mapjoin exception|MAPJOIN异常。通常情况是Mapjoin的小表超过512MB的系统限制造成的。|
|ODPS-0123131:User defined function exception|自定义函数异常|
|ODPS-0123141： Exstore exception|极限存储异常|
|ODPS-0130005:Unknown exception from parser|在顶层捕获所有未知异常|
|ODPS-0130013:Authorization exception|权限不足, 安全检查不通过|
|ODPS-0130025:Failed to I/O|输入输出异常|
|ODPS-0130031:Failed to drop table|删除表时发现源表不存在|
|ODPS-0130041:Statistics exception|统计信息相关异常|
|ODPS-0130051:Exception in sub query|子查询相关异常|
|ODPS-0130061:Invalid table|表不可用|
|ODPS-0130071:Semantic analysis exception - Invalid table alias or column reference|语法解析异常，列名错误，没有找到对应的列。|
|ODPS-0130071:Semantic analysis exception - Invalid column reference|语法解析异常，列引用错误，没有找到对应的列。|
|ODPS-0130071:Semantic analysis exception - Expression not in GROUP BY key|语法解析异常。在Select子句中，读取的列与Group By的列不完全一致。|
|ODPS-0130071:Semantic analysis exception - Partition not found|语法解析异常。没有找到所指定分区值的分区。|
|ODPS-0130071:Semantic analysis exception - SELECT DISTINCT and GROUP BY can not be in the same query|Distinct和Group By不能出现在同一个Select子句中。|
|ODPS-0130071:Semantic analysis exception - Cannot insert into target table because column number/types are different|向目标表插入数据时，源表和目标表的列数量或类型不匹配。|
|ODPS-0130071:Semantic analysis exception - physical plan generation failed: java.lang.RuntimeException: Table\(xxxx\) is full scan with all partitions, please specify partition predicates.|表所属项目禁止了分区表全表扫描，需要指定分区条件|
|ODPS-0130081:Invalid UDF reference|UDF方法签名|
|ODPS-0130091:Invalid parameters|UDF参数不合法|
|ODPS-0130101:Ambiguous data type|数据类型不合法|
|ODPS-0130111:Subquery partition pruning exception|in条件判断语句中的子查询 动态分区优化异常|
|ODPS-0130121:Invalid argument type|非法参数类型。一般情况下，是[内建函数](../cn.zh-CN/用户指南/SQL/内建函数/数学函数.md)接收到的参数类型不正确。|
|ODPS-0130131:Table not found|表不存在。在操作DDL或DML语句时，被操作的表并不存在。|
|ODPS-0130141:Illegal implicit type cast|不允许的隐式类型转换|
|ODPS-0130151:Illegal data type|无效的数据类型|
|ODPS-0130161:Parse exception|语法解析出错|
|ODPS-0130171:Creating view exception|创建视图异常|
|ODPS-0130181:Window function exception|窗口函数异常|
|ODPS-0130191:Invalid column or partition key|非法的列或分区键|
|ODPS-0130201:View not found|视图不存在|
|ODPS-0130211:Table or view already exists|表或视图已存在。出现这种错误一般是由于重复创建同名表或者view，即执行Create Table/View语句时，要创建的Table/View已经存在|
|ODPS-0130221:Invalid number of arguments|参数个法不合法|
|ODPS-0130231:Invalid view|视图状态无效|
|ODPS-0130241:Illegal union operation|无效的union操作。通常情况下是Union两边列的数量及类型不一致造成的。|
|ODPS-0130252:Cartesian product is not allowed|不支持笛卡尔积。MaxCompute在join的on连接中不支持不等值表达式。|
|ODPS-0130261:Invalid schema|非法的元数据|
|ODPS-0130271:Partition does not exist|分区不存在|
|ODPS-0140005:Unknown exception from planner|在顶层捕获所有未知异常|
|ODPS-0140011:Illegal type cast|不允许的显示式类型转换|
|ODPS-0140021:Illegal implicit type cast|不允许的隐式类型转换|
|ODPS-0140031:Invalid column reference|无效的列名或表名引表|
|ODPS-0140041:Invalid UDF reference|UDF名不存在|
|ODPS-0140051:Invalid function|非法函数|
|ODPS-0140061:Invalid parameters|输入参数异常|
|ODPS-0140071:Unsupported operator|不支持的运算符|
|ODPS-0140081:Unsupported join type|小表 \(left\) outer join 大表或者 大表\(right\) outer join小表|
|ODPS-0140091:Unsupported stage type|不支持的执行计划类型|
|ODPS-0140105:Invalid multiple I/O|多路输出冲突|
|ODPS-0140111:Unsupported col type in EXTRACT now|目前EXTRACT不支列类型|
|ODPS-0140125:The path need to read has different storage type| |
|ODPS-0140133：Invalid structure|无法识别数据结构|
|ODPS-0140141：Can not remove operator with more than one children| |
|ODPS-0140151：Can not do topologic sort, the stages is not a DAG|排序算法出现错误|
|ODPS-0140161：Invalid stage| |
|ODPS-0140171:Sandbox violation exception|安全的沙箱模型异常|
|ODPS-0140181:Sql plan exception|由于某种原因导致的SQL作业无法生成执行计划。遇到这种错误，用户可以重试提交作业，多次提交仍失败，可通过工单系统联系我们。|
|ODPS-0123049: buffer overflow|内存溢出，如join里边相同key的数据太多了,内存放不下，需检查数据是否有问题。|
|ODPS-0140178: Internal system failure|系统异常。一般重试可成功。|

