# UNION ALL/UNION \[DISTINCT\] {#concept_m5k_pkb_wdb .concept}

**命令格式如下**：

```
select_statement UNION ALL select_statement;
select_statement UNION [DISTINCT] select_statement;
```

-   UNION ALL：将两个或多个Select操作返回的数据集联合成一个数据集，如果结果有重复行时，会返回所有符合条件的行，不进行重复行的去重处理。
-   UNION \[DISTINCT\]：其中DISTINCT可忽略。将两个或多个Select操作返回的数据集联合成一个数据集，如果结果有重复行时，将进行重复行的去重处理。

UNION ALL示例如下：

```
select * from sale_detail where region = 'hangzhou'
        union all
select * from sale_detail where region = 'shanghai';
```

UNION示例如下：

```
SELECT * FROM src1 UNION SELECT * FROM src2; 
--执行的效果相当于 
SELECT DISTINCT * FROM (SELECT * FROM src1 UNION ALL SELECT * FROM src2) t;
```

**说明：** 

-   union all/union操作对应的各个查询的列个数、名称和类型必须一致。如果列名不一致时，可以使用列的别名加以解决。
-   一般情况下，MaxCompute最多允许256个表的union all/union，超过此限制报语法错误。

关于UNION后LIMIT的语义，如下所示：

UNION后如果有CLUSTER BY、DISTRIBUTE BY、SORT BY、ORDER BY或者LIMIT子句，其作用于前面所有UNION的结果，而不是UNION的最后一个select\_statement。MaxCompute目前在`set odps.sql.type.system.odps2=true;`时，也采用此行为。

**示例如下**：

```
set odps.sql.type.system.odps2=true;
SELECT explode(array(3, 1)) AS (a) UNION ALL SELECT explode(array(0, 4, 2)) AS (a) ORDER BY a LIMIT 3;
```

返回结果如下所示：

```
+------+
| a    |
+------+
| 0    |
| 1    |
| 2    |
+------+
```

