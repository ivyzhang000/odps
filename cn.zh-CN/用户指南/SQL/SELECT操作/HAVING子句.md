# HAVING子句 {#concept_zgy_5kb_wdb .concept}

由于MaxCompute SQL的Where关键字无法与合计函数一起使用，可以采用HAVING字句。

**命令格式如下**：

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

**示例如下**：

比如有一张订单表Orders，包括客户名称（Customer），订单金额（OrderPrice），订单日期（Order\_date），订单号（Order\_id）四个字段。现在希望查找订单总额少于2000的客户。SQL语句如下所示：

```
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```

