# Tunnel命令操作 {#concept_rkf_2wc_5db .concept}

## 功能简介 {#section_hcs_btf_vdb .section}

您可以通过[客户端](../cn.zh-CN/工具及下载/客户端.md)提供的[Tunnel](cn.zh-CN/用户指南/数据上传下载/Tunnel命令操作.md)命令实现原有Dship工具的功能。

Tunnel命令，主要用于数据的上传和下载等功能。主要功能如下：

-   Upload：支持文件或目录（指一级目录）的上传，每一次上传只支持数据上传到一张表或表的一个分区，有分区的表一定要指定上传的分区，多级分区一定要指定到末级分区。

    ```
    tunnel upload log.txt test_project.test_table/p1="b1",p2="b2";
    -- 将log.txt中的数据上传至项目空间test_project的表test_table（二级分区表）中的p1="b1",p2="b2"分区
    tunnel upload  log.txt  test_table --scan=only;
    -- 将log.txt中的数据上传至表 test_table 中。--scan参数表示需要扫描log.txt中的数据是否符合 test_table 的定义，如果不符合报错，并停止上传数据。
    ```

-   Download：只支持下载到单个文件，每一次下载只支持下载一张表或一个分区到一个文件，有分区的表一定要指定下载的分区，多级分区一定要指定到末级分区。

    ```
    tunnel download  test_project.test_table/p1="b1",p2="b2"  test_table.txt;
    -- 将test_project.test_table表（二级分区表）中的数据下载到 test_table.txt 文件中
    ```

-   Resume：因为网络或tunnel服务的原因出错，支持文件或目录的续传。可以继续上一次的数据上传操作，但Resume命令暂时没有对下载操作的支持。

    ```
    tunnel resume;
    ```

-   Show：显示历史任务信息。

    ```
    tunnel show history -n 5;
    --显示前5次上传/下载数据的详细命令
    tunnel show log;
    --显示最后一次上传/下载数据的日志
    ```

-   Purge：清理session目录，默认清理3天内的。

    ```
    tunnel purge 5;
    --清理前5天的日志
    ```


## Tunnel命令使用说明 {#section_g2l_1wf_vdb .section}

Tunnel命令支持在客户端通过help子命令获取帮助信息，每个命令和选择支持短命令格式。

```
odps@ project_name>tunnel help;
    Usage: tunnel <subcommand> [options] [args]
    Type 'tunnel help <subcommand>' for help on a specific subcommand.
Available subcommands:
    upload (u)
    download (d)
    resume (r)
    show (s)
    purge (p)
    help (h)
tunnel is a command for uploading data to / downloading data from ODPS.
```

**参数说明：**

-   upload：上传数据到MaxCompute的表中。
-   download：从MaxCompute的表中下载数据。
-   resume：如果上传数据失败，通过resume命令进行断点续传，目前仅支持上传数据的续传。每次上传、下载数据被称为一个session。在resume命令后指定session id完成续传。
-   show：查看历史运行信息。
-   purge：清理session目录。
-   help：输出tunnel帮助信息。

## Upload {#section_uwr_pwf_vdb .section}

将本地文件的数据导入MaxCompute的表中，以追加模式导入。子命令使用提示：

```
odps@ project_name>tunnel help upload;
usage: tunnel upload [options] <path> <[project.]table[/partition]>
              upload data from local file
 -acp,-auto-create-partition <ARG>   auto create target partition if not
                                     exists, default false
 -bs,-block-size <ARG>               block size in MiB, default 100
 -c,-charset <ARG>                   specify file charset, default ignore.
                                     set ignore to download raw data
 -cp,-compress <ARG>                 compress, default true
 -dbr,-discard-bad-records <ARG>     specify discard bad records
                                     action(true|false), default false
 -dfp,-date-format-pattern <ARG>     specify date format pattern, default
                                     yyyy-MM-dd HH:mm:ss
 -fd,-field-delimiter <ARG>          specify field delimiter, support
                                     unicode, eg \u0001. default ","
 -h,-header <ARG>                    if local file should have table
                                     header, default false
 -mbr,-max-bad-records <ARG>         max bad records, default 1000
 -ni,-null-indicator <ARG>           specify null indicator string,
                                     default ""(empty string)
 -rd,-record-delimiter <ARG>         specify record delimiter, support
                                     unicode, eg \u0001. default "\r\n"
 -s,-scan <ARG>                      specify scan file
                                     action(true|false|only), default true
 -sd,-session-dir <ARG>              set session dir, default
                                     D:\software\odpscmd_public\plugins\ds
                                     hip
 -ss,-strict-schema <ARG>            specify strict schema mode. If false,
                                     extra data will be abandoned and
                                     insufficient field will be filled
                                     with null. Default true
 -te,-tunnel_endpoint <ARG>          tunnel endpoint
    -threads <ARG>                   number of threads, default 1
 -tz,-time-zone <ARG>                time zone, default local timezone:
                                     Asia/Shanghai
Example:
    tunnel upload log.txt test_project.test_table/p1="b1",p2="b2"
```

**参数说明：**

-   -acp：如果不存在，自动创建目标分区，默认关闭。
-   -bs：每次上传至Tunnel的数据块大小，默认100MiB（MiB＝1024\*1024B ） 。
-   -c：指定本地数据文件编码，默认为UTF-8。不设定，默认下载源数据。
-   -cp：指定是否在本地压缩后再上传，减少网络流量，默认开启。
-   -dbr：是否忽略脏数据（多列、少列、列数据类型不匹配等情况）。
    -   值为true时，将全部不符合表定义的数据忽略。
    -   值为false时，若遇到脏数据，则给出错误提示信息，目标表内的原始数据不会被污染。
-   -dfp：DateTime类型数据格式，默认为yyyy-MM-dd HH:mm:ss。
-   -fd：本地数据文件的列分割符，默认为逗号。
-   -h：数据文件是否包括表头，如果为true，则dship会跳过表头从第二行开始上传数据。
-   -mbr：默认情况下，当上传的脏数据超过1000条时，上传动作终止。通过此参数，可以调整可容忍的脏数据量。
-   -ni：NULL数据标志符，默认为“ ”（空字符串）。
-   -rd：本地数据文件的行分割符，默认为\\r\\n。
-   -s：是否扫描本地数据文件，默认值为false。
    -   值为true时，先扫描数据，若数据格式正确，再导入数据。
    -   值为false时，不扫描数据，直接进行数据导入。
    -   值为only时，仅进行扫描本地数据，扫描结束后不继续导入数据。
-   -sd：设置session目录。
-   -te：指定tunnel的Endpoint。
-   -threads：指定threads的数量，默认为1。
-   -tz：指定时区。默认为本地时区：Asia/Shanghai。

**示例如下：**

-   创建目标表，如下所示：

```
CREATE TABLE IF NOT EXISTS sale_detail(
      shop_name     STRING,
      customer_id   STRING,
      total_price   DOUBLE)
PARTITIONED BY (sale_date STRING,region STRING);
```

-   添加分区，如下所示：

    ```
    alter table sale_detail add partition (sale_date='201312', region='hangzhou');
    ```

-   准备数据文件data.txt，其内容如下所示：

    ```
    shop9,97,100
    shop10,10,200
    shop11,11
    ```

    这份文件的第三行数据与sale\_detail的表定义不符。sale\_detail定义了三列，但数据只有两列。

-   导入数据，如下所示：

    ```
    odps@ project_name>tunnel u d:\data.txt sale_detail/sale_date=201312,region=hangzhou -s false
    Upload session: 201506101639224880870a002ec60c
    Start upload:d:\data.txt
    Total bytes:41   Split input to 1 blocks
    2015-06-10 16:39:22     upload block: '1'
    ERROR: column mismatch -,expected 3 columns, 2 columns found, please check data or delimiter
    ```

    由于data.txt中有脏数据，数据导入失败。并给出session id及错误提示信息。

-   数据验证，如下所示：

    ```
    odps@ odpstest_ay52c_ay52> select * from sale_detail where sale_date='201312';
    ID = 20150610084135370gyvc61z5
    +-----------+-------------+-------------+-----------+--------+
    | shop_name | customer_id | total_price | sale_date | region |
    +-----------+-------------+-------------+-----------+--------+
    +-----------+-------------+-------------+-----------+--------+
    ```

    由于有脏数据，数据导入失败，表中无数据。


## Show {#section_wzb_xxf_vdb .section}

显示历史记录。子命令使用提示：

```
odps@ project_name>tunnel help show;
usage: tunnel show history [options]
              show session information
 -n,-number <ARG>   lines
Example:
    tunnel show history -n 5
    tunnel show log
```

**参数说明：**

-n：指定显示行数。

**示例如下：**

```
odps@ project_name>tunnel show  history;
201506101639224880870a002ec60c  failed  'u --config-file /D:/console/conf/odps_config.ini --project odpstest_ay52c_ay52 --endpoint http://service.odps.aliyun.com/api --id UlVxOHuthHV1QrI1 --key 2m4r3WvTZbsNJjybVXj0InVke7UkvR d:\data.txt sale_detail/sale_date=201312,region=hangzhou -s false'
```

**说明：** 201506101639224880870a002ec60c是上节中导入数据失败时的运行ID。

## Resume {#section_wgq_jyf_vdb .section}

修复执行历史记录，仅对上传数据有效。子命令使用提示：

```
odps@  project_name>tunnel help resume;
usage: tunnel resume [session_id] [-force]
              resume an upload session
 -f,-force   force resume
Example:
    tunnel resume
```

**示例如下：**

修改data.txt文件为如下内容：

```
shop9,97,100
shop10,10,200
```

修复执行上传数据，如下所示：

```
odps@ project_name>tunnel resume 201506101639224880870a002ec60c --force;
start resume
201506101639224880870a002ec60c
Upload session: 201506101639224880870a002ec60c
Start upload:d:\data.txt
Resume 1 blocks 
2015-06-10 16:46:42     upload block: '1'
2015-06-10 16:46:42     upload block complete, blockid=1
upload complete, average speed is 0 KB/s
OK
```

**说明：** 201506101639224880870a002ec60c为上传失败的session ID。

数据验证，如下所示：

```
odps@ project_name>select * from sale_detail where sale_date='201312';
 ID = 20150610084801405g0a741z5
 +-----------+-------------+-------------+-----------+--------+
 | shop_name | customer_id | total_price | sale_date | region |
 +-----------+-------------+-------------+-----------+--------+
 | shop9     | 97          | 100.0       | 201312    | hangzhou |
 | shop10    | 10          | 200.0       | 201312    | hangzhou |
 +-----------+-------------+-------------+-----------+--------+
```

## Download {#section_qxw_2zf_vdb .section}

子命令使用提示：

```
odps@ project_name>tunnel help download;
usage: tunnel download [options] <[project.]table[/partition]> <path>
              download data to local file
 -c,-charset <ARG>                 specify file charset, default ignore.
                                   set ignore to download raw data
 -ci,-columns-index <ARG>          specify the columns index(starts from
                                   0) to download, use comma to split each
                                   index
 -cn,-columns-name <ARG>           specify the columns name to download,
                                   use comma to split each name
 -cp,-compress <ARG>               compress, default true
 -dfp,-date-format-pattern <ARG>   specify date format pattern, default
                                   yyyy-MM-dd HH:mm:ss
 -e,-exponential <ARG>             When download double values, use
                                   exponential express if necessary.
                                   Otherwise at most 20 digits will be
                                   reserved. Default false
 -fd,-field-delimiter <ARG>        specify field delimiter, support
                                   unicode, eg \u0001. default ","
 -h,-header <ARG>                  if local file should have table header,
                                   default false
    -limit <ARG>                   specify the number of records to
                                   download
 -ni,-null-indicator <ARG>         specify null indicator string, default
                                   ""(empty string)
 -rd,-record-delimiter <ARG>       specify record delimiter, support
                                   unicode, eg \u0001. default "\r\n"
 -sd,-session-dir <ARG>            set session dir, default
                                   D:\software\odpscmd_public\plugins\dshi
                                   p
 -te,-tunnel_endpoint <ARG>        tunnel endpoint
    -threads <ARG>                 number of threads, default 1
 -tz,-time-zone <ARG>              time zone, default local timezone:
                                   Asia/Shanghai
usage: tunnel download [options] instance://<[project/]instance_id> <path>
              download instance result to local file
 -c,-charset <ARG>                 specify file charset, default ignore.
                                   set ignore to download raw data
 -ci,-columns-index <ARG>          specify the columns index(starts from
                                   0) to download, use comma to split each
                                   index
 -cn,-columns-name <ARG>           specify the columns name to download,
                                   use comma to split each name
 -cp,-compress <ARG>               compress, default true
 -dfp,-date-format-pattern <ARG>   specify date format pattern, default
                                   yyyy-MM-dd HH:mm:ss
 -e,-exponential <ARG>             When download double values, use
                                   exponential express if necessary.
                                   Otherwise at most 20 digits will be
                                   reserved. Default false
 -fd,-field-delimiter <ARG>        specify field delimiter, support
                                   unicode, eg \u0001. default ","
 -h,-header <ARG>                  if local file should have table header,
                                   default false
    -limit <ARG>                   specify the number of records to
                                   download
 -ni,-null-indicator <ARG>         specify null indicator string, default
                                   ""(empty string)
 -rd,-record-delimiter <ARG>       specify record delimiter, support
                                   unicode, eg \u0001. default "\r\n"
 -sd,-session-dir <ARG>            set session dir, default
                                   D:\software\odpscmd_public\plugins\dshi
                                   p
 -te,-tunnel_endpoint <ARG>        tunnel endpoint
    -threads <ARG>                 number of threads, default 1
 -tz,-time-zone <ARG>              time zone, default local timezone:
                                   Asia/Shanghai
Example:
    tunnel download test_project.test_table/p1="b1",p2="b2" log.txt
    tunnel download instance://test_project/test_instance log.txt
```

**参数说明：**

-   -c：本地数据文件编码，默认为UTF-8。
-   -ci：指定列索引（从0）下载，使用逗号分隔。
-   -cn：指定要下载的列名称，使用逗号分隔每个名称。
-   -cp，-compress：指定是否压缩后再下载，减少网络流量，默认开启。
-   -dfp：DateTime类型数据格式，默认为yyyy-MM-dd HH:mm:ss。
-   -e：当下载double值时，如果需要，使用指数函数表示，否则最多保留20位。
-   -fd：本地数据文件的列分割符，默认为逗号。
-   -h：数据文件是否包括表头，如果为true，则dship会跳过表头从第二行开始下载数据。

    **说明：** `-h=true`和`threads>1`即多线程不能一起使用。

-   -limit：指定要下载的文件数量。
-   -ni：NULL数据标志符，默认为“ ”（空字符串）。
-   -rd：本地数据文件的行分割符，默认为\\r\\n。
-   -sd：设置session目录。
-   -te：指定tunnel endpoint。
-   -threads：指定threads的数量，默认为1。
-   -tz：指定时区。默认为本地时区：Asia/Shanghai。

**示例如下：**

下载数据到result.txt文件中，如下所示：

```
$ ./tunnel download sale_detail/sale_date=201312,region=hangzhou result.txt;
    Download session: 201506101658245283870a002ed0b9
    Total records: 2
    2015-06-10 16:58:24     download records: 2
    2015-06-10 16:58:24     file size: 30 bytes
    OK
```

验证result.txt的文件内容，如下所示：

```
shop9,97,100.0
shop10,10,200.0
```

## Purge {#section_l3y_51g_vdb .section}

清除session目录，默认清除距离当前日期3天内的。子命令使用提示：

```
odps@ project_name>tunnel help purge;
usage: tunnel purge [n]
              force session history to be purged.([n] days before, default
              3 days)
Example:
    tunnel purge 5
```

**数据类型说明**：

|类型|描述|
|:-|:-|
|STRING|字符串类型，长度不能超过8MB。|
|BOOLEN|上传值只支持”true”，”false”，”0”，”1”。下载值为true/false且不区分大小写。|
|BIGINT|取值范围\[-9223372036854775807，9223372036854775807\]。|
|DOUBLE|1.有效位数16位 2.上传支持科学计数法表示 3.下载仅使用数字表示 4.最大值：1.7976931348623157E308 5.最小值：4.9E-324 6.无穷大：Infinity 7.无穷小：-Infinity|
|DATETIME|Datetime类型默认支持时区为GMT+8的数据上传，可以通过命令行指定用户数据日期格式的format pattern。|

如果您上传DATETIME类型的数据，需要指定时间日期格式，具体格式请参见SimpleDateFormat。

```
"yyyyMMddHHmmss": 数据格式"20140209101000"
"yyyy-MM-dd HH:mm:ss"（默认）：数据格式"2014-02-09 10:10:00"
"yyyy年MM月dd日": 数据格式"2014年09月01日"
```

**示例如下：**

```
tunnel upload  log.txt  test_table -dfp  "yyyy-MM-dd HH:mm:ss"
```

**空值**：所有数据类型都可以有空值。

-   默认空字符串为空值。
-   可在命令行下通过-null-indicator参数来指定空值的字符串。

```
tunnel upload log.txt test_table -ni "NULL"
```

**字符编码**：您可以指定文件的字符编码，默认为UTF-8。

```
tunnel upload log.txt test_table -c "gbk"
```

**分隔符**：tunnel命令支持您自定义的文件分隔符，行分隔符选项为-record-delimiter，列分隔符选项为-field-delimiter。

分隔符说明如下：

-   支持多个字符的行列分隔符。
-   列分隔符不能够包含行分隔符。
-   转义字符分隔符，在命令行方式下只支持\\r、\\n和\\t。

**示例如下：**

```
tunnel upload log.txt test_table -fd "||" -rd "\r\n"
```

