# 与其他SQL语法的差异 {#concept_fsf_mvf_vdb .concept}

本文将从 SQL 角度，将 MaxCompute SQL 与 Hive、MySQL、Oracle、SQL Server 进行对比，从而为您介绍 MaxCompute 不支持的 DDL 和 DML 语法。

## MaxCompute 不支持的 DDL 语法 {#section_ed4_pvf_vdb .section}

|语法|MaxCompute|Hive|MySql|Oracle|Sql Server|
|:-|:---------|:---|:----|:-----|:---------|
|CREATE TABLE—PRIMARY KEY|N|N|Y|Y|Y|
|CREATE TABLE—NOT NULL|N|N|Y|Y|Y|
|CREATE TABLE—CLUSTER BY|N|Y|N|Y|Y|
|CREATE TABLE—EXTERNAL TABLE|Y（支持OSS和OTS外部表）|Y|N|N|N|
|CREATE TABLE—TEMPORARY TABLE|N|Y|Y|Y|Y （with \#prefix）|
|CREATE INDEX|N|Y|Y|Y|Y|
|VIRTUAl COLUMN|N|N\(only 2 predefined\)|N|Y|Y|

## MaxCompute 不支持的 DML 语法 {#section_ws2_vvf_vdb .section}

|语法|MaxCompute|Hive|MySql|Oracle|Sql Server|
|:-|:---------|:---|:----|:-----|:---------|
|SELECT—recursive CTE|N|N|N|Y|Y|
|SELECT—GROUP BY ROLL UP|N|Y|Y|Y|Y|
|SELECT—GROUP BY CUBE|N|Y|N|Y|Y|
|SELECT—GROUPING SET|N|Y|N|Y|Y|
|SELECT—IMPLICT JOIN|Y|Y|N|Y|Y|
|SELECT—PIVOT|N|N|N|Y|Y|
|SELECT—corelated subquery|N|Y|Y|Y|Y|
|SET OPERATOR—UNION \(disintct\)|Y|Y|Y|Y|Y|
|SET OPERATOR—INTERSECT|N|N|N|Y|Y|
|SET OPERATOR—MINUS|N|N|N|Y|Y（keyword EXCEPT）|
|UPDATE … WHERE|N|Y|Y|Y|Y|
|UPDATE … ORDER BY LIMIT|N|N|Y|N|Y|
|DELETE … WHERE|N|Y|Y|Y|Y|
|DELETE … ORDER BY LIMIT|N|N|Y|N|N|
|ANALYTIC—reusable WINDOWING CLUSUE|N|Y|N|N|N|
|ANALYTIC—CURRENT ROW|N|Y|N|Y|Y|
|ANALYTIC—UNBOUNDED|N|Y|N|Y|Y|
|ANALYTIC—RANGE …|N|Y|N|Y|Y|

