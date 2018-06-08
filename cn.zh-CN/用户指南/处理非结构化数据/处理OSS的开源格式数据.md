# 处理OSS的开源格式数据 {#concept_tlm_34v_ydb .concept}

[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)为您介绍了如何在MaxCompute上访问存储在OSS上的文本、音频、图像等格式的数据。本文将为您介绍对于存储在OSS上的各种流行的开源数据格式（ORC、PARQUET、SEQUENCEFILE、RCFILE、AVRO和TEXTFILE）如何通过非结构化框架在MaxCompute进行处理。

非结构框架会直接调用开源社区的实现来进行开源数据格式的解析，并且与MaxCompute系统无缝对接。

**说明：** 处理OSS的开源格式数据前，需要首先对OSS进行[STS模式授权](cn.zh-CN/用户指南/处理非结构化数据/OSS的STS模式授权.md)。

## 创建External Table {#section_bxs_hsv_ydb .section}

MaxCompute非结构化数据框架通过External Table与各种数据的关联，关联OSS上开源格式数据的External Table建表的DDL语句格式如下：

```
DROP TABLE [IF EXISTS] <external_table>;
CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
[ROW FORMAT SERDE '<serde class>']
STORED AS <file format>
[WITH SERDEPROPERTIES (  'odps.properties.rolearn'='${roleran}'
 [,'name2'='value2',...]
 ) ]
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

**说明：** 该语法格式与Hive的语法相当接近，但需注意以下问题：

-   STORED AS关键字，在该语法格式中不是[普通非结构化外表](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)用的STORED BY关键字，这是目前在读取开源兼容数据时独有的。

    STORED AS后面接的是文件格式名字， 比如ORC/PARQUET/RCFILE/SEQUENCEFILE/TEXTFILE等。

-   外部表的column schemas必须与具体OSS上存储存储数据的schema相符合。
-   ROW FORMAT SERDE非必选选项，只有在使用一些特殊的格式上，比如TEXTFILE时才需要使用。
-   WITH SERDEPROPERTIES当关联OSS权限使用[STS模式授权](cn.zh-CN/用户指南/处理非结构化数据/OSS的STS模式授权.md)时，需要该参数指定odps.properties.rolearn属性，属性值为RAM中具体使用的Role的Arn的信息。

    若不用STS模式，则无需指定该属性，直接在Location传入明文AccessKeyId和AccessKeySecret。

-   Location若关联OSS需使用明文AK，写法如下：

    ```
    LOCATION 'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
    ```


## 关联OSS的PARQUET数据示例 {#section_spx_ttv_ydb .section}

假设有一些PARQUET文件存放在一个OSS路径上，每个文件都是PARQUET格式，存放的schema为16列（4列BigInt、4列Double和8列String）的数据，建表DDL语句如下所示：

```
CREATE EXTERNAL TABLE tpch_lineitem_parquet
(
  l_orderkey bigint,
  l_partkey bigint,
  l_suppkey bigint,
  l_linenumber bigint,
  l_quantity double,
  l_extendedprice double,
  l_discount double,
  l_tax double,
  l_returnflag string,
  l_linestatus string,
  l_shipdate string,
  l_commitdate string,
  l_receiptdate string,
  l_shipinstruct string,
  l_shipmode string,
  l_comment string
)
STORED AS PARQUET
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/parquet_data/';
```

## 关联OSS的TEXTFILE数据分区表示例 {#section_gvy_15v_ydb .section}

如果数据是每行以JSON格式，存储成OSS上TEXTFILE文件，同时数据在OSS通过多个目录组织，这时可以使用MaxCompute分区表和数据关联，建表DDL语句如下所示：

```
CREATE EXTERNAL TABLE tpch_lineitem_textfile
(
  l_orderkey bigint,
  l_partkey bigint,
  l_suppkey bigint,
  l_linenumber bigint,
  l_quantity double,
  l_extendedprice double,
  l_discount double,
  l_tax double,
  l_returnflag string,
  l_linestatus string,
  l_shipdate string,
  l_commitdate string,
  l_receiptdate string,
  l_shipinstruct string,
  l_shipmode string,
  l_comment string
)
PARTITIONED BY (ds string)
ROW FORMAT serde 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/';
```

如果OSS表目录下面的子目录是以Partition Name方式组织，示例如下：

```
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/ds=20170102/'
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/ds=20170103/'
...
```

则可以使用以下DDL语句ADD PARTITION：

```
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170102");
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170103");
```

如果OSS分区目录不是按这种方式组织，或者根本不在表目录下，示例如下：

```
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170102/;
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170103/;
...
```

则可以使用以下DDL语句ADD PARTITION：

```
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170102")
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170102/';
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170103")
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170103/';
...
```

## 读取以及处理OSS的开源格式数据 {#section_ewt_bvv_ydb .section}

对比前文的两个创建外部表示例，可以看出对于不同文件类型，只要简单修改STORED AS后的格式名。在下述示例中将只集中描述对上面PARQUET数据对应的外表（tpch\_lineitem\_parquet）的处理。如果要处理不同的文件类型，只要在DDL创建外表时指定是PARQUET/ORC/TEXTFILE/RCFILE/TEXTFILE即可，处理数据的语句一样。

-   **直接读取以及处理OSS的开源数据**

    创建数据外表进行关联后，直接对外表就可以进行与普通MaxCompute表一样的操作，如下所示：

    ```
    SELECT l_returnflag,
    l_linestatus,
    SUM(l_extendedprice*(1-l_discount)) AS sum_disc_price,
    AVG(l_quantity) AS avg_qty,
    COUNT(*) AS count_order
    FROM tpch_lineitem_parquet
    WHERE l_shipdate <= '1998-09-02'
    GROUP BY
    l_returnflag,
    l_linestatus;
    ```

    外表tpch\_lineitem\_parquet被当作一个普通的内部表一样使用，不同在于MaxCompute内部计算引擎是直接从OSS读取对应的PARQUET数据进行处理。

    前文创建的关联textfile的外部分区表tpch\_lineitem\_textfile，因为使用了`ROW FORMAT` + `STORED AS`，需要手动设置flag（只使用STORED AS，odps.sql.hive.compatible默认为TRUE），再进行读取，否则会有报错：

    ```
    SELECT * FROM tpch_lineitem_textfile LIMIT 1;
    FAILED: ODPS-0123131:User defined function exception - Traceback:
    com.aliyun.odps.udf.UDFException: java.lang.ClassNotFoundException: com.aliyun.odps.hive.wrapper.HiveStorageHandlerWrapper
    --需要手动设置hive兼容flag
    set odps.sql.hive.compatible=true;
    SELECT * FROM tpch_lineitem_textfile LIMIT 1;
    +------------+------------+------------+--------------+------------+-----------------+------------+------------+--------------+--------------+------------+--------------+---------------+----------------+------------+-----------+
    | l_orderkey | l_partkey  | l_suppkey  | l_linenumber | l_quantity | l_extendedprice | l_discount | l_tax      | l_returnflag | l_linestatus | l_shipdate | l_commitdate | l_receiptdate | l_shipinstruct | l_shipmode | l_comment |
    +------------+------------+------------+--------------+------------+-----------------+------------+------------+--------------+--------------+------------+--------------+---------------+----------------+------------+-----------+
    | 5640000001 | 174458698  | 9458733    | 1            | 14.0       | 23071.58        | 0.08       | 0.06       | N            | O            | 1998-01-26 | 1997-11-16   | 1998-02-18    | TAKE BACK RETURN | SHIP       | cuses nag silently. quick |
    +------------+------------+------------+--------------+------------+-----------------+------------+------------+--------------+--------------+------------+--------------+---------------+----------------+------------+-----------+
    ```

    **说明：** 直接使用外表，每次读取数据都需要涉及外部OSS的I/O操作，且MaxCompute系统本身针对内部存储做的许多高性能优化都用不上，如此一来性能上就会有所损失。 因此如果是需要对数据进行反复计算以及对计算的高效性比较敏感的场景，推荐使用下文介绍的用法：先将数据导入MaxCompute内部再进行计算。

-   **将OSS的开源数据导入MaxCompute再进行计算**

    首先创建一个与外部表schema一样的内部表tpch\_lineitem\_internal，然后将OSS上的开源数据导入MaxCompute内部表，以MaxCompute内部数据存储格式进行存储。

    ```
    CREATE TABLE tpch_lineitem_internal LIKE tpch_lineitem_parquet;
    INSERT OVERWRITE TABLE tpch_lineitem_internal
    SELECT * FROM tpch_lineitem_parquet;
    ```

    接下来直接对内部表进行同样的操作：

    ```
    SELECT l_returnflag,
        l_linestatus,
        SUM(l_extendedprice*(1-l_discount)) AS sum_disc_price,
        AVG(l_quantity) AS avg_qty,
        COUNT(*) AS count_order
    FROM tpch_lineitem_internal
    WHERE l_shipdate <= '1998-09-02'
    GROUP BY
        l_returnflag,
        l_linestatus;
    ```

    通过此方式将数据先导入MaxCompute系统进行存储，对同样数据进行计算处理会更高效。


