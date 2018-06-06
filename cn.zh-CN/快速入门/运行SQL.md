# 运行SQL {#concept_tgf_v5y_5db .concept}

大多数用户对SQL的语法并不陌生，简单地说，MaxCompute SQL就是用于查询和分析MaxCompute中的大规模数据。目前SQL的主要功能如下所示：

-   支持各类运算符。
-   通过DDL语句对表、分区以及视图进行管理。
-   通过Select语句查询表中的记录，通过Where语句过滤表中的记录。
-   通过Insert语句插入数据、更新数据。
-   通过等值连接Join操作，支持两张表的关联。支持多张小表的mapjoin。
-   支持通过内置函数和自定义函数来进行计算。
-   支持正则表达式。

本文将为您简单介绍MaxCompute SQL使用中需要注意的问题，不再做操作示例。

**说明：** 

-   MaxCompute SQL不支持事务、索引及Update/Delete等操作，同时MaxCompute的SQL语法与Oracle，MySQL有一定差别，您无法将其他数据库中的SQL语句无缝迁移到MaxCompute上来，更多差异请参见[与其他SQL语法的差异](../cn.zh-CN/用户指南/SQL/附录/与其他SQL语法的差异.md)。
-   在使用方式上，MaxCompute作业提交后会有几十秒到数分钟不等的排队调度，所以适合处理跑批作业，一次作业批量处理海量数据，不适合直接对接需要每秒处理几千至数万笔事务的前台业务系统。
-   关于SQL操作的详细示例，请参见[SQL模块](../cn.zh-CN/用户指南/SQL/SQL概述.md)。

## DDL语句 {#section_kdh_cvy_5db .section}

简单的DDL操作包括创建表、添加分区、查看表和分区信息、修改表、删除表和分区，更多详情请参见[创建/查看/删除表](cn.zh-CN/快速入门/创建/查看/删除表.md)。

**Select语句**

-   group by语句的key可以是输入表的列名，也可以是由输入表的列构成的表达式，不可以是Select语句的输出列。

    ```
    select substr(col2, 2) from tbl group by substr(col2, 2);-- 可以，group by的key可以是输入表的列构成的表达式；
     select col2 from tbl group by substr(col2, 2); -- 不可以，group by的key不在select语句的列中；
     select substr(col2, 2) as c from tbl group by c; -- 不可以，group by的key 不可以是列的别名，即select语句的输出列；
    ```

    之所以有这样的限制，是因为在通常的SQL解析中，group by的操作先于Select操作，因此group by只能接受输入表的列或表达式为key。

-   order by必须与limit连用。
-   sort by前必须加distribute by。
-   order by/sort by/distribute by的key必须是Select语句的输出列，即列的别名。如下所示：

    ```
    select col2 as c from tbl order by col2 limit 100 --不可以，order by的key不是select语句的输出列，即列的别名
      select col2 from tbl order by col2 limit 100; -- 可以，当select语句的输出列没有别名时，使用列名作为别名。
    ```

    之所以有这样的限制，是因为在通常的SQL解析中，order by/sort by/distribute by后于Select操作，因此它们只能接受Select语句的输出列为key。


## Insert语句 { .section}

-   向某个分区插入数据时，分区列不可以出现在Select列表中。

    ```
    insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')  
        select shop_name, customer_id, total_price, sale_date, region  from sale_detail;
      -- 报错返回，sale_date，region为分区列，不可以出现在静态分区的insert语句中。
    ```

-   动态分区插入时，动态分区列必须在Select列表中。

    ```
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region) 
    select shop_name,customer_id,total_price from sale_detail; 
    --失败返回，动态分区插入时，动态分区列必须在select列表中。
    ```


## Join操作 { .section}

-   MaxCompute SQL支持的Join操作类型包括：\{LEFT OUTER|RIGHT OUTER|FULL OUTER|INNER\} JOIN。
-   目前最多支持16个并发Join操作。
-   在mapjoin中，最多支持8张小表的mapjoin。

## Union All {#section_rgd_vvy_5db .section}

Union All可以把多个Select操作返回的结果，联合成一个数据集。它会返回所有的结果，但是不会执行去重。MaxCompute不支持直接对顶级的两个查询结果进行Union操作，需要写成子查询的形式。

**说明：** 

-   Union All连接的两个Select查询语句，两个Select的列个数、列名称、列类型必须严格一致。
-   如果原名称不一致，可以通过别名设置成相同的名称。

## 其他 {#section_nhv_xvy_5db .section}

-   MaxCompute SQL目前最多支持128个并发Union操作。
-   最多支持128个并发insert overwrite/into操作。

MaxCompute SQL的更多限制请参见[SQL限制项汇总](../cn.zh-CN/用户指南/SQL/SQL限制项汇总.md)。

## SQL优化示例 {#section_osj_bwy_5db .section}

-   **Join语句中Where条件的位置**

    当两个表进行Join操作时，主表的Where限制可以写在最后，但从表分区限制条件不要写在Where条件中，建议写在ON条件或者子查询中。主表的分区限制条件可以写在Where条件中（最好先用子查询过滤）。示例如下：

    ```
    select * from A join (select * from B where dt=20150301)B on B.id=A.id where A.dt=20150301； 
    select * from A join B on B.id=A.id where B.dt=20150301； --不允许 
    select * from (select * from A where dt=20150301)A join (select * from B where dt=20150301)B on B.id=A.id；
    ```

    第二个语句会先Join，后进行分区裁剪，数据量变大，性能下降。在实际使用过程中，应该尽量避免第二种用法。

-   **数据倾斜**

    产生数据倾斜的根本原因是有少数Worker处理的数据量远远超过其他Worker处理的数据量，从而导致少数Worker的运行时长远远超过其他的平均运行时长，从而导致整个任务运行时间超长，造成任务延迟。

    更多数据倾斜优化的详情请参见[计算长尾调优](../cn.zh-CN/最佳实践/计算长尾调优.md)。

    -   **Join造成的数据倾斜**

        造成Join数据倾斜的原因是Join on的key分布不均匀。假设还是上述示例语句，现在将大表A和小表B进行Join操作，运行如下语句：

        ```
        select * from A join B on A.value= B.value;
        ```

        此时，复制logview的链接并打开webcosole页面，双击执行Join操作的fuxi job，可以看到此时在\[Long-tails\]区域有长尾，表示数据已经倾斜了。如下图所示：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11952/1560_zh-CN.PNG)

        此时您可通过如下方法进行优化：

        -   由于表B是个小表并且没有超过512MB，您可将上述语句优化为mapjoin语句再执行，语句如下：

            ```
            select /*+ MAPJOIN(B) */ * from A join B on A.value= B.value;
            ```

        -   您也可将倾斜的key用单独的逻辑来处理，例如经常发生两边的key中有大量null数据导致了倾斜。则需要在Join前先过滤掉null的数据或者补上随机数，然后再进行Join。示例如下：

            ```
            select * from A join B
            on case when A.value is null then concat('value',rand() ) else A.value end = B.value;
            ```

    在实际场景中，如果您知道数据倾斜了，但无法获取导致数据倾斜的key信息，那么可以使用一个通用的方案，查看数据倾斜。如下所示：

    ```
    
    例如：select * from a join b on a.key=b.key; 产生数据倾斜。 
    您可以执行： 
    ```sql
    select left.key, left.cnt * right.cnt from 
    (select key, count(*) as cnt from a group by key) left 
    join
    (select key, count(*) as cnt from b group by key) right
    on left.key=right.key;
    ```

    查看key的分布，可以判断a join b时是否会有数据倾斜。

-   **group by倾斜**

    造成group by倾斜的原因是group by的key分布不均匀。

    假设表A内有两个字段（key，value），表内的数据量足够大，并且key的值分布不均，运行语句如下所示：

    ```
    select key,count(value) from A group by key;
    ```

    当表中的数据足够大时，您会在webcosole页面看见长尾。若想解决这个问题，您需要在执行SQL前设置防倾斜的参数，设置语句为`set odps.sql.groupby.skewindata=true`。

-   **错误使用动态分区造成的数据倾斜**

    动态分区的SQL，在MaxCompute中会默认增加一个Reduce，用来将相同分区的数据合并在一起。这样做的好处，如下所示：

    -   可减少MaxCompute系统产生的小文件，使后续处理更快速。
    -   可避免一个Worker输出文件很多时占用内存过大。
    但也正是因为这个Reduce的引入，导致分区数据如果有倾斜的话，会发生长尾。因为相同的数据最多只会有10个Worker处理，所以数据量大，则会发生长尾。示例如下：

    ```
    
    insert overwrite table A2 partition(dt) 
    select
    split_part(value,'\t',1) as field1,
    split_part(value,'\t',2) as field2, 
    dt 
    from A 
    where dt='20151010';
    ```

    这种情况下，没有必要使用动态分区，所以可以改为如下语句：

    ```
    
    insert overwrite table A2 partition(dt='20151010') 
    select
    split_part(value,'\t',1) as field1,
    split_part(value,'\t',2) as field2
    from A 
    where dt='20151010';
    ```

-   **窗口函数的优化**

    如果您的SQL语句中用到了窗口函数，一般情况下每个窗口函数会形成一个Reduce作业。如果窗口函数略多，那么就会消耗资源。在某些特定场景下，窗口函数是可以进行优化的。

    -   窗口函数over后面要完全相同，相同的分组和排序条件。
    -   多个窗口函数在同一层SQL执行。
    符合上述两个条件的窗口函数会合并为一个Reduce执行。SQL示例如下所示：

    ```
    
    select
    rank()over(partition by A order by B desc) as rank,
    row_number()over(partition by A order by B desc) as row_num
    from MyTable;
    ```

-   **子查询改Join**

    例如有一个子查询，如下所示：

    ```
    SELECT * FROM table_a a WHERE a.col1 IN (SELECT col1 FROM table_b b WHERE xxx);
    ```

    当此语句中的table\_b子查询返回的col1的个数超过1000个时，系统会报错为`records returned from subquery exceeded limit of 1000`。此时您可以使用Join语句来代替，如下所示：

    ```
    SELECT a.* FROM table_a a JOIN (SELECT DISTINCT col1 FROM table_b b WHERE xxx) c ON (a.col1 = c.col1)
    ```

    **说明：** 

    -   如果没用Distinct，而子查询c返回的结果中有相同的col1的值，可能会导致a表的结果数变多。
    -   因为Distinct子句会导致查询全落到一个Worker里，如果子查询数据量比较大的话，可能会导致查询比较慢。
    -   如果已经从业务上控制了子查询里的col1不可能会重复，比如查的是主键字段，为了提高性能，可以把Distinct去掉。

