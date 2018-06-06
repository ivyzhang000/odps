# LIKE字符匹配 {#concept_gdj_btf_vdb .concept}

在 LIKE 匹配时，`%`表示匹配任意多个字符，`\_`表示匹配单个字符，如果要匹配`%`或`\_`本身，则要对其进行转义，`\%`匹配字符`%`， `\\_`匹配字符`\_`。

```
    'abcd' like 'ab%' -- true
    'abcd' like 'ab\%' -- true
    'ab\_cde' like 'ab\\_c%'; -- true
```

**说明：** 关于字符串的字符集，目前 MaxCompute SQL 支持 UTF-8 的字符集，如果数据是以其它格式编码，可能计算出的结果不正确。

