# 访问OTS非结构化数据 {#concept_dgw_n2b_wdb .concept}

表格存储（Table Store）是构建在阿里云飞天分布式系统之上的NoSQL数据存储服务，提供海量结构化数据的存储和实时访问。您可以通过[TableStore文档](https://help.aliyun.com/document_detail/27280.html)对其进行了解。

MaxCompute与TableStore是两个独立的大数据计算和存储服务，所以两者之间的网络必须保证连通性。MaxCompute公共云服务访问TableStore存储时，推荐您使用TableStore**私网**地址，也就是host名以ots-internal.aliyuncs.com作为结尾的地址，例如：tablestore://odps-ots-dev.cn-shanghai.ots-internal.aliyuncs.com。

前文为您介绍了如何[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)，本文将进一步为您介绍如何将来自TableStore（OTS）的数据纳入MaxCompute上的计算生态，实现多种数据源之间的无缝连接。

TableStore与MaxCompute都有其自身的类型系统。在MaxCompute处理TableStore数据时，两者之间的类型对应关系如下所示：

|MaxCompute Type|TableStore Type|
|:--------------|:--------------|
|STRING|STRING|
|BIGINT|INTEGER|
|DOUBLE|DOUBLE|
|BOOLEAN|BOOLEAN|
|BINARY|BINARY|

## STS模式授权 {#section_spx_rrb_wdb .section}

MaxCompute计算服务访问Table Store数据需要有一个安全的授权通道。在此问题上，MaxCompute结合了阿里云的访问控制服务（RAM）和令牌服务（STS）来实现对数据的安全访问。

您可以通过以下两种方式授予权限：

-   当MaxCompute和Table Store的Owner是同一个账号时，登录阿里云账号后，[单击此处完成一键授权](https://ram.console.aliyun.com/?spm=5176.100239.blogcont281191.24.uJg9dR#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D)。
-   自定义授权
    1.  首先在RAM控制台中授予MaxCompute访问Table Store的权限。

        登录 [RAM控制台](https://ram.console.aliyun.com/#/overview)（若MaxCompute和Table Store不是同一个账号，此处需由Table Store账号登录进行授权），创建角色，角色名叫 AliyunODPSDefaultRole或AliyunODPSRoleForOtherUser 。

    2.  修改策略内容设置，如下所示：

        ```
        --当MaxCompute和Table Store的Owner是同一个账号  
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        --当MaxCompute和Table Store的Owner不是同一个账号
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "MaxCompute的Owner云账号的UID@odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        ```

        **说明：** 您可单击右上角的登录账号，进入账号管理页面查看云账号的UID。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12076/2844_zh-CN.png)

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12076/2845_zh-CN.jpg)

    3.  编辑该角色的授权策略AliyunODPSRolePolicy，如下所示：

        ```
        {
        "Version": "1",
        "Statement": [
        {
         "Action": [
           "ots:ListTable",
           "ots:DescribeTable",
           "ots:GetRow",
           "ots:PutRow",
           "ots:UpdateRow",
           "ots:DeleteRow",
           "ots:GetRange",
           "ots:BatchGetRow",
           "ots:BatchWriteRow",
           "ots:ComputeSplitPointsBySize"
         ],
         "Resource": "*",
         "Effect": "Allow"
        }
        ]
        }
        --还可自定义其他权限
        ```

    4.  将权限AliyunODPSRolePolicy授权给该角色。

## 创建外部表 {#section_dh2_ksb_wdb .section}

MaxCompute通过创建外部表，把对TableStore表数据的描述引入到MaxCompute的meta系统内部后，即可轻松实现对TableStore数据的处理。本节将以下述示例为例，来为您说明 axCompute对接TableStore的一些概念和实现。

建外部表语句如下：

```
DROP TABLE IF EXISTS ots_table_external;
CREATE EXTERNAL TABLE IF NOT EXISTS ots_table_external
(
odps_orderkey bigint,
odps_orderdate string,
odps_custkey bigint,
odps_orderstatus string,
odps_totalprice double
)
STORED BY 'com.aliyun.odps.TableStoreStorageHandler' -- (1)
WITH SERDEPROPERTIES ( -- (2)
'tablestore.columns.mapping'=':o_orderkey,:o_orderdate,o_custkey, o_orderstatus,o_totalprice', -- (3)
'tablestore.table.name'='ots_tpch_orders' -- (4)
'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
)
LOCATION 'tablestore://odps-ots-dev.cn-shanghai.ots-internal.aliyuncs.com'; -- (5)
```

语句说明如下所示：

-   com.aliyun.odps.TableStoreStorageHandler是MaxCompute内置的处理TableStore数据的StorageHandler，定义了MaxCompute和TableStore的交互，相关逻辑由MaxCompute实现。
-   SERDEPROPERITES是提供参数选项的接口，在使用TableStoreStorageHandler时，有两个必须指定的选项，分别是下面将会介绍的tablestore.columns.mapping 、 tablestore.table.name和odps.properties.rolearn。
    -   tablestore.columns.mapping选项：必选项，用来描述MaxCompute将访问的Table Store表的列，包括主键和属性列。
        -   以`:`打头的用来表示Table Store主键，例如此语句中的`:o_orderkey`和`:o_orderdate`，其他的均为属性列。
        -   Table Store支持1-4个主键，主键类型为String、Integer和Binary，其中第一个主键为分区键。
        -   在指定映射时，您必须提供指定Table Store表的所有主键，对于属性列则没有必要全部提供，可以只提供需要通过MaxCompute来访问的属性列。
    -   tablestore.table.name：需要访问的Table Store表名。如果指定的Table Store表名错误（不存在），则会报错，MaxCompute不会主动去创建Table Store表。
    -   odps.properties.rolearn中的信息是RAM中AliyunODPSDefaultRole的Arn信息。您可以通过RAM控制台中的**角色详情**进行获取。
-   LOCATION clause：用来指定Table Storeinstance名字、endpoint等具体信息。这里的Table Store数据的安全访问建立在前文介绍的RAM/STS授权的前提上。


如果您想要查看创建好的外部表结构信息，可以执行如下语句：

```
desc extended <table_name>;

```

在返回的信息里，除了跟内部表一样的基础信息外，Extended Info包含外部表StorageHandler 、Location等信息。

## 查询外部表 {#section_dcl_ssb_wdb .section}

创建External Table后，Table Store的数据便引入到了MaxCompute生态中，即可通过正常的MaxCompute SQL语法访问Table Store数据，如下所示：

```
SELECT odps_orderkey, odps_orderdate, SUM(odps_totalprice) AS sum_total
FROM ots_table_external
WHERE odps_orderkey > 5000 AND odps_orderkey < 7000 AND odps_orderdate >= '1996-05-03' AND odps_orderdate < '1997-05-01'
GROUP BY odps_orderkey, odps_orderdate
HAVING sum_total> 400000.0;
```

由上可见，使用常见的MaxCompute SQL语法，访问Table Store的所有细节由MaxComput 内部处理。这包括在列名的选择上，比如上述SQL中，使用的列名是odps\_orderkey，odps\_totalprice等，而不是原始Table Store中的主键名o\_orderkey或属性列名o\_totalprice，因为在创建External Table的DDL语句中，已经做了对应的mapping。当然您也可根据自己的需求在创建External Table时选择保留原始的TableStore主键/列名。

如果需要对一份数据做**多次计算**，相较每次从Table Store去远程读数据，有个更高效的办法是先一次性把需要的数据导入到MaxCompute内部成为一个MaxCompute（内部）表，示例如下：

```
CREATE TABLE internal_orders AS
SELECT odps_orderkey, odps_orderdate, odps_custkey, odps_totalprice
FROM ots_table_external
WHERE odps_orderkey > 5000 ;
```

现在internal\_orders就是一个MaxCompute表了，也拥有所有MaxCompute内部表的特性，包括高效的压缩列存储数据格式、完整的内部宏数据以及统计信息等。同时因为存储在MaxCompute内部，访问速度会比访问外部的Table Store更快，尤其适用于需要进行多次计算的热点数据。

## MaxCompute导出数据到Table Store {#section_gy2_xsb_wdb .section}

**说明：** MaxCompute不会主动创建外部的Table Store表，所以在对Table Store表进行数据输出之前，必须保证该表已经在Table Store上创建过（否则将报错）。

根据上面的操作，您已创建了外部表ots\_table\_external来打通MaxCompute与Table Storeb数据表ots\_tpch\_orders的链路，同时还有一份存储在MaxCompute内部表internal\_orders的数据，现在希望对internal\_orders中的数据进行一定处理后再写回 Table Store，可通过对外部表做**INSERT OVERWITE TABLE**操作来实现，如下所示：

```
INSERT OVERWRITE TABLE ots_table_external
SELECT odps_orderkey, odps_orderdate, odps_custkey, CONCAT(odps_custkey, 'SHIPPED'), CEIL(odps_totalprice)
FROM internal_orders;
```

对于Table Store这种KV数据的NoSQL存储介质，从MaxCompute的输出将只影响相对应主键所在的行，比如示例中只影响所有odps\_orderkey + odps\_orderdate这两个主键值能对应行上的数据。而且在这些Tabele Store行上面，也只会去更新在创建External Table（ots\_table\_external）时指定的属性列，而不会去修改未在External Table中出现的数据列。更多详情请参见[MaxCompute访问TableStore（OTS）数据](https://yq.aliyun.com/articles/69314)。

