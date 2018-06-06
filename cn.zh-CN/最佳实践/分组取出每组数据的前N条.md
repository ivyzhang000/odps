# 分组取出每组数据的前N条 {#concept_wnd_z4c_5db .concept}

本文将为您介绍如何对数据进行分组，取出每组数据的前 N 条数据。

## 示例数据 {#section_qkr_fpc_5db .section}

目前的数据，如下表所示：

|empno|ename|job|sal|
|:----|:----|:--|:--|
|7369|SMITH|CLERK|800.0|
|7876|SMITH|CLERK|1100.0|
|7900|JAMES|CLERK|950.0|
|7934|MILLER|CLERK|1300.0|
|7499|ALLEN|SALESMAN|1600.0|
|7654|MARTIN|SALESMAN|1250.0|
|7844|TURNER|SALESMAN|1500.0|
|7521|WARD|SALESMAN|1250.0|

## 实现方法 {#section_ykr_fpc_5db .section}

您可以通过以下两种方法实现：

-   取出每条数据的行号，再用 where 语句进行过滤。

    ```
    
    SELECT * FROM (
      SELECT empno
      , ename
      , sal
      , job
      , ROW_NUMBER() OVER (PARTITION BY job ORDER BY sal) AS rn
      FROM emp
    ) tmp
    WHERE rn < 10;
    ```

-   使用 UDTF 实现 Split 函数。

    详情请参见 [此文](https://yq.aliyun.com/articles/67084) 中最后的示例。这个例子可以更迅速地判断当前的序号，如果是已经超过预定的条数（比如 10 条），便不做处理了，从而提高计算效率。


