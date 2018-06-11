# 更新表中的数据（INSERT OVERWRITE/INTO） {#concept_yzd_ndb_wdb .concept}

**命令格式如下**：

```
INSERT OVERWRITE|INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [(col1,col2 ...)]
select_statement
FROM from_statement;
```

**说明：** 

-   MaxCompute的Insert语法与通常使用的MySQL或Oracle的Insert语法有差别，在insert overwrite|into后需要加入table关键字，不是直接使用tablename。
-   当Insert的目标表是分区表时，指定分区值`[PARTITION (partcol1=val1, partcol2=val2 …)]`语法中不允许使用函数等表达式。
-   目前INSERT OVERWRITE还不支持指定插入列的功能，暂时只能用INSERT INTO。

在MaxCompute SQL处理数据的过程中，Insert overwrite/into用于将计算的结果保存目标表中。

Insert into与Insert overwrite的区别是：Insert into会向表或表的分区中追加数据，而Insert overwrite会在向表或分区中插入数据前清空表中的原有数据。

在使用MaxCompute处理数据的过程中，Insert overwrite/into是最常用到的语句，它们会将计算的结果保存到一个表中，以供下一步计算使用。比如计算sale\_detail表中不同地区的销售额，操作如下：

```
create table sale_detail_insert like sale_detail;
alter table sale_detail_insert add partition(sale_date='2013', region='china');
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select shop_name, customer_id, total_price from sale_detail;
```

**说明：** 在进行Insert更新数据操作时，源表与目标表的对应关系依赖于在select子句中列的顺序，而不是表与表之间列名的对应关系，下面的SQL语句仍然是合法的：

```
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select customer_id, shop_name, total_price from sale_detail;
    -- 在创建sale_detail_insert表时，列的顺序为：
    -- shop_name string, customer_id string, total_price bigint
    -- 而从sale_detail向sale_detail_insert插入数据是，sale_detail的插入顺序为：
    -- customer_id, shop_name, total_price
    -- 此时，会将sale_detail.customer_id的数据插入sale_detail_insert.shop_name
    -- 将sale_detail.shop_name的数据插入sale_detail_insert.customer_id
```

向某个分区插入数据时，分区列不允许出现在select列表中：

```
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
        select shop_name, customer_id, total_price, sale_date, region  from sale_detail;
    -- 报错返回，sale_date，region 为分区列，不允许出现在静态分区的 insert 语句中。
```

同时，partition的值只能是常量，不可以出现表达式。以下用法是非法的：

```
insert overwrite table sale_detail_insert partition (sale_date=datepart('2016-09-18 01:10:00', 'yyyy') , region='china')
        select shop_name, customer_id, total_price from sale_detail;
```

