# 视图操作 {#concept_wjt_sj1_wdb .concept}

## 创建视图 {#section_jpr_4z1_wdb .section}

**创建视图的语法格式，如下所示：**

```
CREATE [OR REPLACE] VIEW [IF NOT EXISTS] view_name
    [(col_name [COMMENT col_comment], ...)]
    [COMMENT view_comment]
    [AS select_statement]
```

**说明：** 

-   创建视图时，必须有对视图所引用表的读权限。
-   视图只能包含一个有效的select语句。
-   视图可以引用其它视图，但不能引用自己，也不能循环引用。
-   不允许向视图写入数据，例如使用insert into或者insert overwrite操作视图。
-   当建好视图后，如果视图的引用表发生了变更，有可能导致视图无法访问，例如删除被引用表。您需要自己维护引用表及视图之间的对应关系。
-   如果没有指定if not exists，在视图已经存在时用create view会导致异常。这种情况可以用create or replace view来重建视图，重建后视图本身的权限保持不变。

**示例如下：**

```
create view if not exists sale_detail_view
(store_name, customer_id, price, sale_date, region)
comment 'a view for table sale_detail'
as select * from sale_detail;
```

## 删除视图 {#section_ynd_xz1_wdb .section}

**删除视图的语法格式，如下所示：**

```
DROP VIEW [IF EXISTS] view_name;
```

**说明：** 如果视图不存在且没有指定if exists，则报错。

**示例如下：**

```
DROP VIEW IF EXISTS sale_detail_view;
```

## 重命名视图 {#section_y2p_11b_wdb .section}

**重命名视图的语法格式，如下所示：**

```
ALTER VIEW view_name RENAME TO new_view_name;
```

**说明：** 如果已存在同名视图，则报错。

**示例如下：**

```
create view if not exists sale_detail_view
        (store_name, customer_id, price, sale_date, region)
        comment 'a view for table sale_detail'
        as select * from sale_detail;
    alter view sale_detail_view rename to market;
```

