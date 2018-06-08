# MapReduce常见错误 {#concept_znv_42w_tdb .concept}

|错误信息|触发条件|
|:---|:---|
|ODPS-0710005：Exception occurred when connecting to meta store|连接meta store时发生错误|
|ODPS-0710015 MR task encountered meta store exception|运行MR task时发生meta store类错误|
|ODPS-0710025：Get partitions from meta store error|从meta store获取partition信息时发生错误|
|ODPS-0720001： Invalid table name format|用户指定非法的表名格式|
|ODPS-0720015：PartKeys size do not match partVals size|table schema partVal不一致|
|ODPS-0720021： Column does not exist|指定不存在的列|
|ODPS-0720031 PARSER Cache resource between comma must not be empty|指定的resource list在逗号之间为空|
|ODPS-0720041：Resource not found|指定了不存在的resouce|
|ODPS-0720051：Resource table should not be a view|指定的resource table是view|
|ODPS-0720061： Table or partition not found for resource|指定了不存在的table或table partition作为resource|
|ODPS-0720071： Total size of cache resources is too big|制定的resource的总个数或者总大小超过限制（默认256个/512M）|
|ODPS-0720081： Job has not specified mapper class|job没有指定mapper class|
|ODPS-0720091：jColumn duplicate|ob input指定了重复的column|
|ODPS-0720101： Input table should not be a view|input table是view|
|ODPS-0720111：Output table should not be a view|output table是view|
|ODPS-0720121：Invalid table partSpec|制定的table partSpec非法|
|ODPS-0720131：Invalid multiple output|非法的multi output（包括duplicate label,same output tables or partitions）|
|ODPS-0720141： Memory value out of bound|指定了不在\[256M, 12G\]范围内的memory value,|
|ODPS-0720151： Cpu value out of bound|指定了不在\[50, 800\]范围内的cpu value|
|ODPS-0720161： Invalid max attempts value|指定了不在（0，6）范围内的odps.mapred.map/reduce.max.attempts|
|ODPS-0720171 ： Invalid IO sort buffer|指定的io sort buffer不在范围（64M,odps.mapred.map/reduce.memory）|
|ODPS-0720181：Classpath resource between comma must not be empty|classpath的逗号之间为空|
|ODPS-0720191：Invalid input split mode|指定了非法的split mod|
|ODPS-0720201：Invalid map split size|指定的odps.mapred.map.min/max.split.size不合法|
|ODPS-0720211：Invalid number of map tasks|指定的odps.mapred.map.tasks不在1,100000|
|ODPS-0720221：Invalid max splits number|指定了非法的max sample split|
|ODPS-0720231：Job input not set|job没有指定input|
|ODPS-0720241：Num of map instance is too big|map instance数大于odps.mapred.max.map.tasks|
|ODPS-0720251： Num of reduce instance is invalid|reduce instance数不在\[0,odps.mapred.reduce.tasks\]范围|
|ODPS-0720261：Invalid partition value|指定了非法的partition value|
|ODPS-0720271： Allow no input is conflict to split mode|用户设置了setallowNoInput\(true\)的同时设置了不同于ALLOW\_NO\_INPUT的split mode|
|ODPS-0720281: Invalid partitition format|partition格式非法|
|ODPS-0720291:Invalid description json|input description json格式非法|
|ODPS-0720301:Too many job input|指定的input数超过odps.mapred.map.max.input.num（默认1024）|
|ODPS-0720311:Invalid output label|指定的output label格式非法|
|ODPS-0720321: Too many job output|指定的outout数超过odps.mapred.max.output.num（默认256）|
|ODPS-0720331: Too many cache resources|指定了超过odps.mapred.job.max.cache.resource.num的resource数|
|ODPS-0730005:MR job internal error|发生MR task中不受用户控制internal error|

