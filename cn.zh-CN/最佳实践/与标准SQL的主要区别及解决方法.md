# 与标准SQL的主要区别及解决方法 {#concept_tcb_h12_5db .concept}

本文将从习惯使用关系型数据库 SQL 用户的实践角度出发，列举用户在使用 MaxCompute SQL 时比较容易遇见的问题。具体的 MaxCompute SQL 语法请参见[SQL 概述](../cn.zh-CN/用户指南/SQL/SQL概述.md)。

## MaxCompute SQL 基本区别 {#section_zt4_pb2_5db .section}

**应用场景**

-   不支持事物（没有 commit 和 rollback，建议代码具有等幂性支持重跑，不推荐使用 Insert Into，推荐 Insert Overwrite 写入数据）。

-   不支持索引和主外键约束。

-   不支持自增字段和默认值。如果有默认值，请在数据写入时自行赋值。

**表分区**

-   单表支持 6 万个分区。

-   [一次查询输入的分区数不能大于 1 万](https://help.aliyun.com/document_detail/43152.html)，否则执行会报错。另外如果是 2 级分区且查询时只根据 2 级分区进行过滤，总的分区数大于 1 万也可能导致报错。

-   [一次查询输出的分区数不能大于 2048](https://help.aliyun.com/document_detail/44226.html)。


**精度**

-   Double 类型因为存在精度问题，不建议在关联时候进行直接等号关联两个 Double 字段。一个比较推荐的做法是把两个数做下减法，如果差距小于一个预设的值就认为是相同，比如 abs\(a1- a2\) < 0.000000001。

-   目前产品上已经支持高精度的类型 Decimal。但如果有更高精度要求的，可以先把数据存为 String 类型，然后使用 UDF 来实现对应的计算。


**数据类型转换**

-   为防止出现各种预期外的错误，如果有 2 个不同的字段类型需要做 Join，建议您先把类型转好后再 Join，同时更容易维护代码。

-   关于日期型和字符串的隐式转换。在需要传入日期型的函数里如果传入一个字符串，字符串和日期类型的转换根据 yyyy-mm-dd hh:mi:ss 格式进行转换。如果是其他格式请参见 [日期函数 \> TO\_DATE](../cn.zh-CN/用户指南/SQL/内建函数/日期函数.md)。


## DDL 的区别及解法 { .section}

**表结构**

-   不能修改分区列列名，只能修改分区列对应的值。具体分区列和分区的区别请参见 [常见问题](https://help.aliyun.com/document_detail/40278.html)。

-   支持增加列，但是不支持删除列以及修改列的数据类型，请参见 [SQL 常见问题](https://help.aliyun.com/document_detail/40292.html)。


## DML 的区别及解法 {#section_bz4_hd2_5db .section}

**INSERT**

-   语法上最直观的区别是：Insert into/overwrite 后面有个关键字 **Table**。

-   数据插入表的字段映射不是根据 Select 的别名做的，而是根据 Select 的字段的顺序和表里的字段的顺序。


**UPDATE/DELETE**

-   目前不支持 Update/Delete 语句，如果有需要请参见 [更新和删除数据](https://help.aliyun.com/document_detail/40275.html)。

**SELECT**

-   [输入表的数量不能超过 16 张](https://help.aliyun.com/document_detail/44309.html)。

-   Group by 查询中的 Select 字段，要么是 Group By 的分组字段，要么需要使用聚合函数。从逻辑角度理解，如发现一个非分组列同一个 Group By Key 中的数据有多条，不使用聚合函数的话就没办法展示。

-   MaxCompute不支持Group by cube（group by with rollup） ，可以通过union来模拟，比如 `select k1, null, sum(v) from t group by k1 union all select k1, k2, sum(v) from t group by k1, k2;`。

 **子查询** 

子查询必须要有别名。建议查询都带别名。

**IN/NOT IN**

-   关于 In/Not In,Exist/Not Exist，后面的子查询数据量不能超过 1000 条，解决办法请参见 [如何使用Not In](https://help.aliyun.com/document_detail/40282.html)。如果业务上已经保证了子查询返回结果的唯一性，可以考虑去掉 Distinct，从而提升查询性能。

**SQL 返回 10000 条**

-   MaxCompute 限制了单独执行 Select 语句时返回的数据条数，具体配置请参见[其他操作](../cn.zh-CN/用户指南/常用命令/其他操作.md)，设置上限为 1 万。如果需要查询的结果数据条数很多，请参见 [如何获取所有数据](https://help.aliyun.com/document_detail/40333.html)，配合 Tunnel 命令获取全部数据。

**MAPJOIN**

-   Join 不支持笛卡尔积，也就是 Join 必须要用 on 设置关联条件。如果有一些小表需要做广播表，需要用 Mapjoin Hint。详情请参见 [如何解决Join报错](https://help.aliyun.com/document_detail/40268.html)。

**ORDER BY**

-   **Order By 后面需要配合 Limit n 使用**。如果希望做很大的数据量的排序，甚至需要做全表排序，可以把这个 N 设置的很大。不过请谨慎使用，因为无法使用到分布式系统的优势，可能会有性能问题。详情请参见 [MaxCompute 查询数据的排序](https://help.aliyun.com/document_detail/40302.html)。

**UNION ALL**

-   参与 UNION ALL 运算的所有列的数据类型、列个数、列名称必须完全一致，否则抛异常。

-   UNION ALL 查询外面需要再嵌套一层子查询。


