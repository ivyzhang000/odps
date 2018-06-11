# VALUES {#concept_uhn_rdb_wdb .concept}

通常在业务测试阶段，需要给一个小数据表准备些基本数据，您可以通过INSERT … VALUES的方法快速对测试表写入一些测试数据。

**说明：** 

目前INSERT OVERWRITE还不支持这种指定插入列的功能，暂时只能用INSERT INTO。

**命令格式如下**：

```
INSERT  INTO  TABLE  tablename 
[PARTITION (partcol1=val1, partcol2=val2 ...)][co1name1,colname2...] 
[VALUES (col1_value,col2_value,...),(col1_value,col2_value,...),...]
```

**示例一**：

```
drop table if exists srcp;
create table if not exists srcp (key string ,value bigint) partitioned by (p string);
insert into table srcp partition (p='abc') values ('a',1),('b',2),('c',3);
```

Insert…values语句执行成功后，查询表srcp分区p=abc，结果如下：

```
+-----+------------+---+
| key | value      | p |
+-----+------------+---+
| a   | 1          | abc |
| b   | 2          | abc |
| c   | 3          | abc |
+-----+------------+---+
```

当表有很多列，而准备数据的时候希望只插入部分列的数据，此时可以使用插入列表功能。

**示例二：**

```
drop table if exists srcp;
create table if not exists srcp (key string ,value bigint) partitioned by (p string);
insert into table srcp partition (p)(key,p) values ('d','20170101'),('e','20170101'),('f','20170101');
```

Insert…values语句执行成功后，查询表srcp分区p=20170101，结果如下：

```
+-----+------------+---+
| key | value      | p |
+-----+------------+---+
| d   | NULL       | 20170101 |
| e   | NULL       | 20170101 |
| f   | NULL       | 20170101 |
+-----+------------+---+
```

对于在values中没有制定的列，可以看到取缺省值为NULL。插入列表功能不一定和values一起用，对于Insert into…select…，同样可以使用。

Insert…values有一个限制：values必须是常量，但是有时候希望在插入的数据中进行一些简单的运算，此时可以使用MaxCompute的values table功能，详情见示例三。

**示例三：**

```
drop table if exists srcp;
create table if not exists srcp (key string ,value bigint) partitioned by (p string);
insert into table srcp partition (p) select concat(a,b), length(a)+length(b),'20170102' from  values ('d',4),('e',5),('f',6) t(a,b);
```

其中的values \(…\), \(…\) t \(a, b\)相当于定义了一个名为t，列为a，b的表，类型为（a string，b bigint），其中的类型从values列表中推导。这样在不准备任何物理表的时候，可以模拟一个有任意数据的，多行的表，并进行任意运算。

Insert…values语句执行成功后，查询表srcp分区p=20170102，结果如下：

```
+-----+------------+---+
| key | value      | p |
+-----+------------+---+
| d4  | 2          | 20170102 |
| e5  | 2          | 20170102 |
| f6  | 2          | 20170102 |
+-----+------------+---+
```

**说明：** 

-   values只支持常量不支持函数，类似ARRAY复杂类型目前无法构造对应的常量，可以写为如下语句：

    ```
    insert into table srcp (p ='abc') select 'a',array('1', '2',
            '3');
    ```

    可达到一样的效果。

-   通过values写入DATETIME、TIMESTAMP类型，需要在values中指定类型名称，如下所示：

    ```
    insert into table srcp (p ='abc') values (datetime'2017-11-11
                00:00:00',timestamp'2017-11-11 00:00:00.123456789');
    ```


实际上，values表并不限于在Insert语句中使用，任何DML语句都可以使用。

还有一种values表的特殊形式，如下所示：

```
select abs(-1), length('abc'), getdate();
```

如上述语句所示，可以不写from语句，直接执行select，只要select的表达式列表不用任何上游表数据就可以。其底层实现为从一个1行，0列的匿名values表选取。这样，在希望测试一些函数，比如自己的UDF等时，便不用再手工创建DUAL表。

