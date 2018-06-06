# 快速掌握SQL写法 {#concept_jxd_4qw_5db .concept}

本文通过课程实践的方式，为您介绍 MaxCompute SQL，让您快速掌握 SQL 的写法，并清楚 MaxCompute SQL 和标准 SQL 的区别，请结合 [MaxCompute SQL 基础文档](../DNODPS1898901/ZH-CN_TP_11987_V1.dita) 进行阅读。

## 数据集准备 {#section_cxt_3rw_5db .section}

这里选择大家比较熟悉的 Emp/Dept 表做为数据集。为方便大家操作，特提供相关的 MaxCompute 建表语句和数据文件（[emp 表数据文件](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/51009/cn_zh/1489374509403/emp%E6%95%B0%E6%8D%AE.txt)，[dept 表数据文件](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/51009/cn_zh/1488265664915/dept%E8%A1%A8%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6.txt)），您可自行在 MaxCompute 项目上创建表并上传数据。

创建 emp 表的 DDL 语句，如下所示：

```

CREATE TABLE IF NOT EXISTS emp (
  EMPNO string ,
  ENAME string ,
  JOB string ,
  MGR bigint ,
  HIREDATE datetime ,
  SAL double ,
  COMM double ,
  DEPTNO bigint );
```

创建 dept 表的 DDL 语句，如下所示：

```

CREATE TABLE IF NOT EXISTS dept (
  DEPTNO bigint ,
  DNAME string ,
  LOC string);
```

## SQL操作 {#section_hcy_nvw_5db .section}

**初学 SQL 常遇到的问题点**

-   使用 Group by，那么 Select 的部分要么是分组项，要么就得是聚合函数。

-   Order by 后面必须加 Limit n。

-   Select 表达式中不能用子查询，可以改写为 Join。

-   Join 不支持笛卡尔积，以及 MapJoin 的用法和使用场景。

-   Union all 需要改成子查询的格式。

-   In/Not in 语句对应的子查询只能有一列，而且返回的行数不能超过 1000，否则也需要改成 Join。


**编写 SQL 进行解题**

**题目一：列出至少有一个员工的所有部门。**

为了避免数据量太大的情况下导致 **常遇问题点** 中的第 6 点，您需要使用 Join 进行改写。如下所示：

```

SELECT d.*
FROM dept d
JOIN (
    SELECT DISTINCT deptno AS no
    FROM emp
) e
ON d.deptno = e.no;
```

**题目二：列出薪金比 SMITH 多的所有员工。**

MapJoin 的典型场景，如下所示：

```

SELECT /*+ MapJoin(a) */ e.empno
    , e.ename
    , e.sal
FROM emp e
JOIN (
    SELECT MAX(sal) AS sal
    FROM `emp`
    WHERE `ENAME` = 'SMITH'
) a
ON e.sal > a.sal;
```

**题目三：列出所有员工的姓名及其直接上级的姓名。**

非等值连接，如下所示：

```

SELECT a.ename
    , b.ename
FROM emp a
LEFT OUTER JOIN emp b
ON b.empno = a.mgr;
```

**题目四：列出最低薪金大于 1500 的各种工作。**

Having 的用法，如下所示：

```

SELECT emp.`JOB`
    , MIN(emp.sal) AS sal
FROM `emp`
GROUP BY emp.`JOB`
HAVING MIN(emp.sal) > 1500;
```

**题目五：列出在每个部门工作的员工数量、平均工资和平均服务期限。**

时间处理上有很多好用的内建函数，如下所示：

```

SELECT COUNT(empno) AS cnt_emp
    , ROUND(AVG(sal), 2) AS avg_sal
    , ROUND(AVG(datediff(getdate(), hiredate, 'dd')), 2) AS avg_hire
FROM `emp`
GROUP BY `DEPTNO`;
```

**题目六： 列出每个部门的薪水前3名的人员的姓名以及他们的名次（Top n 的需求非常常见）。**

SQL 语句如下所示：

```

SELECT *
FROM (
  SELECT deptno
    , ename
    , sal
    , ROW_NUMBER() OVER (PARTITION BY deptno ORDER BY sal DESC) AS nums
  FROM emp
) emp1
WHERE emp1.nums < 4;
```

**题目七： 用一个 SQL 写出每个部门的人数、CLERK（办事员）的人数占该部门总人数占比**。

SQL 语句如下所示：

```

SELECT deptno
    , COUNT(empno) AS cnt
    , ROUND(SUM(CASE 
      WHEN job = 'CLERK' THEN 1
      ELSE 0
    END) / COUNT(empno), 2) AS rate
FROM `EMP`
GROUP BY deptno;
```

