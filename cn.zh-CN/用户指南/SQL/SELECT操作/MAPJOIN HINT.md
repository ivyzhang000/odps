# MAPJOIN HINT {#concept_bf5_tkb_wdb .concept}

当一个大表和一个或多个小表做Join时，可以使用MapJoin，性能比普通的Join要快很多。MapJoin 的基本原理为：在小数据量情况下，SQL会将您指定的小表全部加载到执行Join操作的程序的内存中，从而加快Join的执行速度。

**说明：** 当您使用 MapJoin 时，要注意以下问题：

-   left outer join的左表必须是大表。
-   right outer join的右表必须是大表。
-   inner join左表或右表均可以作为大表。
-   full outer join不能使用MapJoin。
-   MapJoin支持小表为子查询。
-   使用MapJoin时，需要引用小表或是子查询时，需要引用别名。
-   在MapJoin中，可以使用不等值连接或者使用or连接多个条件。
-   目前，MaxCompute在MapJoin中最多支持指定8张小表，否则报语法错误。
-   如果使用MapJoin，则所有小表占用的内存总和不得超过512MB。由于MaxCompute是压缩存储，因此小表在被加载到内存后，数据大小会急剧膨胀。此处的512MB限制是加载到内存后的空间大小。
-   多个表Join时，最左边的两个表不能同时是MapJoin的表。

**示例如下**：

```
select /* + mapjoin(a) */
        a.shop_name,
        b.customer_id,
        b.total_price
    from shop a join sale_detail b
    on a.shop_name = b.shop_name;
```

MaxCompute SQL不支持在普通Join的on条件中使用不等值表达式，or逻辑等复杂的Join条件，但是在MapJoin中可以进行如上操作。

**示例如下**：

```
select /*+ mapjoin(a) */
        a.total_price,
        b.total_price
    from shop a join sale_detail b
    on a.total_price < b.total_price or a.total_price + b.total_price < 500;
```

