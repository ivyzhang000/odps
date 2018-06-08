# 生命周期操作 {#concept_axp_nj1_wdb .concept}

## 修改表的生命周期 {#section_u1v_dz1_wdb .section}

MaxCompute提供数据生命周期管理功能，以方便您释放存储空间，简化回收数据的流程。

**修改表的生命周期属性的语法格式，如下所示：**

```
ALTER TABLE table_name SET lifecycle days;
```

**说明：** 

-   days参数为生命周期时间，只接受正整数，单位为天。
-   如果表table\_name是非分区表，自最后一次数据被修改开始计算，经过days天后数据仍未被改动，则此表无需您干预，将会被MaxCompute自动回收（类似drop table操作）。
-   在MaxCompute中，每当表的数据被修改后，表的LastDataModifiedTime将会被更新，因此，MaxCompute会根据每张表的LastDataModifiedTime以及lifecycle的设置来判断是否要回收此表。
-   如果table\_name是分区表，则根据各分区的LastDataModifiedTime判断该分区是否该被回收。
-   不同于非分区表，分区表的最后一个分区被回收后，该表不会被删除。
-   生命周期只能设定到表级别，不能再分区级设置生命周期。
-   创建表时即可指定生命周期。

**示例如下：**

```
create table test_lifecycle(key string) lifecycle 100;
 -- 新建test_lifecycle表，生命周期为100天。
 alter table test_lifecycle set lifecycle 50;
 -- 修改test_lifecycle表，将生命周期设为50天。
```

## 禁止生命周期 {#section_zbv_kz1_wdb .section}

某些情况下，部分特定的分区不希望被生命周期功能自动回收掉，比如一个月的月初或双十一期间的数据，此时您可以禁止该分区被生命周期功能回收。

**禁止生命周期的语法格式，如下所示：**

```
ALTER TABLE table_name [partition_spec] ENABLE|DISABLE LIFECYCLE;
```

示例如下：

```
ALTER TABLE trans PARTITION(dt='20141111') DISABLE LIFECYCLE;
```

