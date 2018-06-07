# SQL实现多行数据转一条 {#concept_qxm_dcc_5db .concept}

本文将为您介绍，如何使用 SQL 实现多条数据压缩为一条。

## 场景示例 {#section_yhj_3lc_5db .section}

以下表数据为例：

|class|gender|name|
|:----|:-----|:---|
|1|M|LiLei|
|1|F|HanMM|
|1|M|Jim|
|2|F|Kate|
|2|M|Peter|

**场景一**

根据需求，常见场景如下：

|class|names|
|:----|:----|
|1|LiLei,HanMM,Jim|
|2|Kate,Peter|

类似这样使用某个分隔符做字符串拼接，可以使用如下语句：

```
SELECT class, wm_concat(',', name) FROM students GROUP BY class;
```

**场景二**

另外一种常见需求，如下所示：

|class|cnt\_m|cnt\_f|
|:----|:-----|:-----|
|1|2|1|
|2|1|1|

类似这样转多列的需求，可以使用如下语句：

```

SELECT 
class
,SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS cnt_m
,SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS cnt_f
FROM students
GROUP BY class;
```

