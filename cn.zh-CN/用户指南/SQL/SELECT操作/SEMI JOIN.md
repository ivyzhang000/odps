# SEMI JOIN {#concept_lls_skb_wdb .concept}

MaxCompute支持SEMI JOIN（半连接）。SEMI JOIN中，右表只用来过滤左表的数据而不出现在结果集中。支持LEFT SEMI JOIN和LEFT ANTI JOIN两种语法。

## LEFT SEMI JOIN {#section_kjz_pwb_wdb .section}

当Join条件成立时，返回左表中的数据。也就是mytable1中某行的Id在mytable2的所有Id中出现过，此行就保留在结果集中。

**示例如下**：

```
SELECT * from mytable1 a LEFT SEMI JOIN mytable2 b on a.id=b.id;
```

只会返回mytable1中的数据，只要mytable1的Id在mytable2的Id中出现。

## LEFT ANTI JOIN {#section_c1y_twb_wdb .section}

当Join条件不成立时，返回左表中的数据。也就是mytable1中某行的Id在mytable2的所有Id中没有出现过，此行便保留在结果集中。

**示例如下**：

```
SELECT * from mytable1 a LEFT ANTI JOIN mytable2 b on a.id=b.id;
```

只会返回mytable1中的数据，只要mytable1的Id在mytable2的Id没有出现。

