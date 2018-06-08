# Select语序 {#concept_jsv_mkb_wdb .concept}

按照Select语法格式书写的Select语句，实际上的逻辑执行顺序与标准的书写语序实际并不相同，如下语句：

```
SELECT key, max(value) FROM src t WHERE value > 0 GROUP BY key HAVING sum(value) > 100 ORDER BY key LIMIT 100;
```

实际上的逻辑执行顺序是`FROM->WHERE->GROUY BY->HAVING->SELECT->ORDER BY->LIMIT`。order by中只能引用Select列表中生成的列，而不是访问From的源表中的列。Having可以访问的是group by key和聚合函数。Select的时候，如果有group by，便只能访问group key和聚合函数，而不是From中源表中的列。

为了避免混淆，MaxCompute支持以执行顺序书写查询语句，例如上面的语句可以写为：

```
FROM src t WHERE value > 0 GROUP BY key HAVING sum(value) > 100 SELECT key, max(value) ORDER BY key LIMIT 100;
```

