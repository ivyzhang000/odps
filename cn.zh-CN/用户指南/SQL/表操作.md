# 表操作 {#concept_l3j_w31_wdb .concept}

## 创建表 {#section_qqp_st1_wdb .section}

**创建表的语法格式，如下所示：**

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
    [(col_name data_type [COMMENT col_comment], ...)]
    [COMMENT table_comment]
    [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
    [STORED BY StorageHandler] -- 仅限外部表
    [WITH SERDEPROPERTIES (Options)] -- 仅限外部表
    [LOCATION OSSLocation];-- 仅限外部表
    [LIFECYCLE days]
    [AS select_statement]
    CREATE TABLE [IF NOT EXISTS] table_name
    LIKE existing_table_name
```

**说明如下：**

-   创建表时，如果不指定if not exists选项而存在同名表，则返回出错。若指定此选项，则无论是否存在同名表，即使原表结构与要创建的目标表结构不一致，均返回成功。已存在的同名表的元信息不会被改动。
-   表名与列名均对大小写不敏感，不能有特殊字符，只能用英文的a-z，A-Z及数字和下划线\_，且以字母开头，名称的长度不超过128字节。
-   单表的列定义个数最多1200个。
-   数据类型：Bigint、Double、Boolean、Datetime、Decimal和String等，MaxCompute2.0版本扩展了很多[数据类型](../cn.zh-CN/产品简介/基本概念/数据类型.md)。

    **说明：** 要使用新数据类型（Tinyint、Smallint、 Int、 Float、Varchar、TIMESTAMP BINARY），需在建表语句前加上set语句`set odps.sql.type.system.odps2=true;`，并与建表语句一起提交执行。

-   Partitioned by指定表的[分区](../cn.zh-CN/产品简介/基本概念/分区.md)字段，目前支持Tinyint、Smallint、 Int、 Bigint、Varchar和String类型。

    分区值不允许有双字节字符（如中文），必须是以英文字母a-z，A-Z开始后可跟字母数字，名称的长度不超过128字节。允许的字符包括：空格、冒号（:）、下划线（\_）、美元符（$）、井号（\#）、点（.），感叹号（!）和（@），出现其他字符行为未定义，例如（\\t）、（\\n）、（/）等。当利用分区字段对表进行分区时，新增分区、更新分区内数据和读取分区数据均不需要做全表扫描，可以提高处理效率。

-   一张表最多允许60000个分区，单表的分区层次不能超过6级。
-   注释内容是长度不超过1024字节的有效字符串。
-   lifecycle表的生命周期，单位：天。create table like语句不会复制源表的生命周期属性。
-   有关外部表的更多详情请参见[处理非结构化数据](cn.zh-CN/用户指南/处理非结构化数据.md)。

**示例如下：**

假设创建表sale\_detail来保存销售记录，该表使用销售时间（sale\_date）和销售区域 （region）作为分区列，建表语句如下所示：

```
create table if not exists sale_detail(
 shop_name     string,
 customer_id   string,
 total_price   double)
 partitioned by (sale_date string,region string);
 -- 创建一张分区表 sale_detail
```

通过`create table…as select…`语句创建表，并在建表的同时将数据复制到新表中，如下所示：

```
create table sale_detail_ctas1 as
select * from sale_detail;
```

此时，如果sale\_detail中存在数据，上面的示例会将sale\_detail的数据全部复制到sale\_detail\_ctas1表中。

**说明：** 此处sale\_detail是一张分区表，而通过`create table…as select…`语句创建的表不会复制分区属性，只会把源表的分区列作为目标表的一般列处理，即sale\_detail\_ctas1是一个含有5列的非分区表。

在`create table…as select…`语句中，如果在select子句中使用常量作为列的值，建议指定列的名字，如下所示：

```
create table sale_detail_ctas2 as
        select shop_name,
            customer_id,
            total_price,
            '2013' as sale_date,
            'China' as region
        from sale_detail;
```

如果不加列的别名，如下所示：

```
create table sale_detail_ctas3 as
        select shop_name,
            customer_id,
            total_price,
            '2013',
            'China'
        from sale_detail;
```

则创建的表sale\_detail\_ctas3的第四、五列类似于`_c5`、`_c6`。

如果希望源表和目标表具有相同的表结构，可以尝试使用`create table...like`操作，如下所示：

```
create table sale_detail_like like sale_detail;
```

此时，`sale_detail_like`的表结构与`sale_detail`完全相同。除生命周期属性外，列名、列注释以及表注释等均相同。但`sale_detail`中的数据不会被复制到`sale_detail_like`表中。

## 查看表信息 {#section_cm5_mv1_wdb .section}

**查看表信息的语法格式，如下所示：**

```
desc <table_name>;
desc extended <table_name>;--查看外部表信息
```

**示例如下：**

-   假设查看上述示例中表sale\_detail的信息，可输入如下命令：

    ```
    desc sale_detail;
    ```

    结果如下所示：

    ```
    odps@ $odps_project>desc sale_detail;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$maojing.mj@alibaba-inc.com | Project: $odps_project
                      |
    | TableComment:
       |
    +--------------------------------------------------------------------+
    | CreateTime:               2017-06-28 15:05:17
       |
    | LastDDLTime:              2017-06-28 15:05:17
       |
    | LastModifiedTime:         2017-06-28 15:05:17
       |
    +--------------------------------------------------------------------+
    | InternalTable: YES      | Size: 0
       |
    +--------------------------------------------------------------------+
    | Native Columns:
       |
    +--------------------------------------------------------------------+
    | Field           | Type       | Label | Comment
       |
    +--------------------------------------------------------------------+
    | shop_name       | string     |       |
       |
    | customer_id     | string     |       |
       |
    | total_price     | double     |       |
       |
    +--------------------------------------------------------------------+
    | Partition Columns:
       |
    +--------------------------------------------------------------------+
    | sale_date       | string     |
       |
    | region          | string     |
       |
    +--------------------------------------------------------------------+
    OK
    ```

-   假设查看上述示例表sale\_detail\_like中的信息，可输入如下命令：

    ```
    desc sale_detail_like
    ```

    结果如下所示：

    ```
    odps@ $odps_project>desc sale_detail_like;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$maojing.mj@alibaba-inc.com | Project: $odps_project
                      |
    | TableComment:
       |
    +--------------------------------------------------------------------+
    | CreateTime:               2017-06-28 15:42:17
       |
    | LastDDLTime:              2017-06-28 15:42:17
       |
    | LastModifiedTime:         2017-06-28 15:42:17
       |
    +--------------------------------------------------------------------+
    | InternalTable: YES      | Size: 0
       |
    +--------------------------------------------------------------------+
    | Native Columns:
       |
    +--------------------------------------------------------------------+
    | Field           | Type       | Label | Comment
       |
    +--------------------------------------------------------------------+
    | shop_name       | string     |       |
       |
    | customer_id     | string     |       |
       |
    | total_price     | double     |       |
       |
    +--------------------------------------------------------------------+
    | Partition Columns:
       |
    +--------------------------------------------------------------------+
    | sale_date       | string     |
       |
    | region          | string     |
       |
    +--------------------------------------------------------------------+
    OK
    ```


由上可见，除生命周期属性外，sale\_detail\_like的其他属性（字段类型，分区类型等）均与sale\_detail完全一致。查看表信息的更多详情请参见[Describe Table](cn.zh-CN/用户指南/常用命令/表操作.md)。

如果您查看表sale\_detail\_ctas1的信息，会发现sale\_date、region两个字段仅会作为普通列存在，而不是表的分区。

## 删除表 {#section_xvw_1w1_wdb .section}

**删除表的语法格式，如下所示：**

```
DROP TABLE [IF EXISTS] table_name;
```

**说明：** 

-   如果不指定if exists选项而表不存在，则返回异常。若指定此选项，无论表是否存在，皆返回成功。
-   删除外部表时，OSS上的数据不会被删除。

**示例如下：**

```
create table sale_detail_drop like sale_detail;
    drop table sale_detail_drop;
    --若表存在，成功返回；若不存在，异常返回；
    drop table if exists sale_detail_drop2;
    --无论是否存在 sale_detail_drop2 表，均成功返回。
```

## 重命名表 {#section_k4t_2w1_wdb .section}

**重命名表的语法格式，如下所示：**

```
ALTER TABLE table_name RENAME TO new_table_name;
```

**说明：** 

-   rename操作仅修改表的名字，不改动表中的数据。
-   如果已存在与new\_table\_name同名表，则报错。
-   如果table\_name不存在，则报错。

**示例如下：**

```
create table sale_detail_rename1 like sale_detail;
alter table sale_detail_rename1 rename to sale_detail_rename2;
```

## 修改表的注释 {#section_qnj_ny1_wdb .section}

**修改表的注释的语法格式，如下所示：**

```
ALTER TABLE table_name SET COMMENT 'tbl comment';
```

**说明：** 

-   table\_name必须是已存在的表。
-   comment最长1024字节。

**示例如下：**

```
alter table sale_detail set comment 'new coments for table sale_detail';
```

通过MaxCompute的desc命令可以查看表中comment的修改，详情请参见[常用命令\>表操作](cn.zh-CN/用户指南/常用命令/表操作.md)中的Describe Table。

## 修改表的修改时间 {#section_gxl_5y1_wdb .section}

MaxCompute SQL提供touch操作用来修改表的LastDataModifiedTime，可将表的LastDataModifiedTime修改为当前时间。

**修改表的修改时间的语法格式，如下所示：**

```
ALTER TABLE table_name TOUCH;
```

**说明：** 

-   table\_name不存在，则报错返回。
-   此操作会改变表的LastDataModifiedTime的值，此时，MaxCompute会认为表的数据有变动，生命周期的计算会重新开始。

## 清空非分区表里的数据 {#section_zpx_1z1_wdb .section}

将指定的非分区表中的数据清空，该命令不支持分区表。对于分区表，可以用`ALTER TABLE table_name DROP PARTITION`的方式将分区里的数据清除。

**清空非分区表里的数据的语法格式，如下所示：**

```
TRUNCATE TABLE table_name;
```

