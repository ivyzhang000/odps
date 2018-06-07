# 访问OSS非结构化数据 {#concept_tzd_tlg_vdb .concept}

本文将带您实现在MaxCompute上轻松访问OSS的数据，关于处理非结构化数据的原理性介绍请参见[前言](cn.zh-CN/用户指南/处理非结构化数据/前言.md#)。

## STS模式授予权限 {#section_uv5_1fb_wdb .section}

MaxCompute需要直接访问OSS的数据，前提需要将OSS的数据相关权限赋给MaxCompute的访问账号，您可通过以下两种方式授予权限：

-   **当MaxCompute和OSS的owner是同一个账号时**，可以直接登录阿里云账号后，[点击此处完成一键授权](https://ram.console.aliyun.com/?spm=5176.100239.blogcont281191.24.uJg9dR#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D)。
-   自定义授权。
    1.  首先需要在[RAM](https://www.aliyun.com/product/ram)中授予MaxCompute访问OSS的权限。登录 [RAM控制台](https://ram.console.aliyun.com/#/overview)（若MaxCompute和OSS不是同一个账号，此处需由OSS账号登录进行授权），通过控制台中的 [角色管理](https://ram.console.aliyun.com/#/role/list) 创建角色 ，角色名如AliyunODPSDefaultRole或AliyunODPSRoleForOtherUser。如下图所示：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12074/5234_zh-CN.png)

    2.  修改角色策略内容设置，如下所示：

        ```
        --当MaxCompute和OSS的Owner是同一个账号
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
        --当MaxCompute和OSS的Owner不是同一个账号
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "MaxCompute的Owner云账号id@odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        ```

    3.  授予角色访问OSS必要的权限**AliyunODPSRolePolicy** 。如下所示：

        ```
        {
        "Version": "1",
        "Statement": [
        {
         "Action": [
           "oss:ListBuckets",
           "oss:GetObject",
           "oss:ListObjects",
           "oss:PutObject",
           "oss:DeleteObject",
           "oss:AbortMultipartUpload",
           "oss:ListParts"
         ],
         "Resource": "*",
         "Effect": "Allow"
        }
        ]
        }
        --可自定义其他权限
        ```

    4.  将权限**AliyunODPSRolePolicy**授权给该角色。

## 内置extractor访问OSS数据 {#section_ukx_nfb_wdb .section}

访问外部数据源时，需要用户自定义不同的Extractor，同时您也可以使用MaxCompute内置的Extractor，来读取按照约定格式存储的[OSS](https://www.aliyun.com/product/oss)数据。只需要创建一个外部表，便可把这张表作为源表进行查询。

假设有一份CSV数据存在[OSS](https://www.aliyun.com/product/oss)上，endpoint为`oss-cn-shanghai-internal.aliyuncs.com`，bucket为`oss-odps-test`，数据文件的存放路径为/demo/vehicle.csv。

## 创建外部表 {#section_b23_tfb_wdb .section}

创建外部表，语句如下：

```
CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_csv_external
(
vehicleId int,
recordId int,
patientId int,
calls int,
locationLatitute double,
locationLongtitue double,
recordTime string,
direction string
)
STORED BY 'com.aliyun.odps.CsvStorageHandler' -- (1)
WITH SERDEPROPERTIES (
 'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
) -- (2)
LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/'; -- (3)(4)
```

上述语句，说明如下：

-   `com.aliyun.odps.CsvStorageHandler`是内置的处理CSV格式文件的`StorageHandler`，它定义了如何读写CSV文件。您只需指明这个名字，相关逻辑已经由系统实现。
-   `odps.properties.rolearn`中的信息是RAM中`AliyunODPSDefaultRole`的`Arn`信息。您可以通过RAM控制台中的[角色详情](https://ram.console.aliyun.com/#/role/detailAliyunODPSDefaultRole/info)获取。
-   LOCATION必须指定一个OSS目录，默认系统会读取这个目录下所有的文件。
    -   建议您使用OSS提供的内网域名，否则将产生OSS流量费用。
    -   建议您存放OSS数据的区域对应您开通MaxCompute的区域。由于MaxCompute只有在部分区域部署，我们不承诺跨区域的数据连通性。
    -   OSS的连接格式为`oss://oss-cn-shanghai-internal.aliyuncs.com/Bucket名称/目录名称/`。目录后不要加文件名称，如下的集中用法都是错误的：

        ```
        http://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/  -- 不支持http连接
        https://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/ -- 不支持https连接
        oss://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo -- 连接地址错误
        oss://oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/vehicle.csv  -- 不必指定文件名
        ```

-   外部表只是在系统中记录了与OSS目录的关联，当Drop这张表时，对应的`LOCATION`数据不会被删除。

    更多有关外部表的说明请参见[DDL语句](cn.zh-CN/用户指南/SQL/DDL语句.md)。


如果想查看创建好的外部表结构信息，可以执行语句：

```
desc extended <table_name>;
```

在返回的信息里，除了跟内部表一样的基础信息外，Extended Info包含外部表StorageHandler 、Location等信息。

## 查询外部表 {#section_ljx_ggb_wdb .section}

外部表创建成功后，便可如同普通表一样使用这个外部表。假设`/demo/vehicle.csv`数据如下：

```
1,1,51,1,46.81006,-92.08174,9/14/2014 0:00,S
1,2,13,1,46.81006,-92.08174,9/14/2014 0:00,NE
1,3,48,1,46.81006,-92.08174,9/14/2014 0:00,NE
1,4,30,1,46.81006,-92.08174,9/14/2014 0:00,W
1,5,47,1,46.81006,-92.08174,9/14/2014 0:00,S
1,6,9,1,46.81006,-92.08174,9/14/2014 0:00,S
1,7,53,1,46.81006,-92.08174,9/14/2014 0:00,N
1,8,63,1,46.81006,-92.08174,9/14/2014 0:00,SW
1,9,4,1,46.81006,-92.08174,9/14/2014 0:00,NE
1,10,31,1,46.81006,-92.08174,9/14/2014 0:00,N
```

执行如下SQL语句：

```
select recordId, patientId, direction from ambulance_data_csv_external where patientId > 25;
```

**说明：** 目前外部表只能通过MaxCompute SQL操作，MaxCompute MapReduce目前无法操作外部表。

这条语句会提交一个作业，调用内置csv extractor，从OSS读取数据进行处理。输出结果如下：

```
+------------+------------+-----------+
| recordId   | patientId  | direction |
+------------+------------+-----------+
| 1          | 51         | S         |
| 3          | 48         | NE        |
| 4          | 30         | W         |
| 5          | 47         | S         |
| 7          | 53         | N         |
| 8          | 63         | SW        |
| 10         | 31         | N         |
+------------+------------+-----------+
```

## 读取gzip压缩的CSV/TSV数据 {#section_qd2_dgr_zdb .section}

MaxCompute目前只支持通过内置extractor读取OSS上gzip压缩的CSV/TSV数据。与读取非压缩文件相比，主要区别是SERDEPROPERTIES指定的属性。

创建外部表的代码示例，如下所示：

```
CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_csv_external
(
vehicleId bigint,
recordId bigint,
patientId bigint,
calls bigint,
locationLatitute double,
locationLongtitue double,
recordTime string,
direction string
)
STORED BY 'com.aliyun.odps.CsvStorageHandler'
WITH SERDEPROPERTIES (
 'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
 [,'odps.text.option.gzip.input.enabled'='true']
 [,'name3'='value3']
)
LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/Demo/SampleData/CSV/AmbulanceData/';
```

目前SERDEPROPERTIES还支持的属性项，如下所示：

|属性名|属性值|默认值|说明|
|:--|:--|:--|:-|
|odps.text.option.gzip.input.enabled|true/false|false|打开/关闭读压缩|
|odps.text.option.gzip.output.enabled|true/false|false|打开/关闭写压缩|
|odps.text.option.header.lines.count|非负整数|0|跳过文本文件头N行|
|odps.text.option.null.indicator|字符串|空字符串|在解析或者写出NULL值时代表NULL的字符串|
|odps.text.option.ignore.empty.lines|true/false|true|是否忽略空行|
|odps.text.option.encoding|UTF-8/UTF-16/US-ASCII|UTF-8|指定文本的字符编码|

**说明：** 

当关联OSS压缩数据的外部表，读（select）、写（insert）操作也需要进行，那么创建这个外部表时，需要同时设置odps.text.option.gzip.input.enabled和odps.text.option.gzip.output.enabled两个属性为true。

## 自定义Extractor访问OSS {#section_lbc_ngb_wdb .section}

当OSS中的数据格式比较复杂，内置的Extractor无法满足需求时，需要自定义Extractor来读取OSS文件中的数据。

例如有一个txt数据文件，并不是CSV格式，记录之间的列通过`|`分隔。比如`/demo/SampleData/CustomTxt/AmbulanceData/vehicle.csv`数据如下：

```
1|1|51|1|46.81006|-92.08174|9/14/2014 0:00|S
1|2|13|1|46.81006|-92.08174|9/14/2014 0:00|NE
1|3|48|1|46.81006|-92.08174|9/14/2014 0:00|NE
1|4|30|1|46.81006|-92.08174|9/14/2014 0:00|W
1|5|47|1|46.81006|-92.08174|9/14/2014 0:00|S
1|6|9|1|46.81006|-92.08174|9/14/2014 0:00|S
1|7|53|1|46.81006|-92.08174|9/14/2014 0:00|N
1|8|63|1|46.81006|-92.08174|9/14/2014 0:00|SW
1|9|4|1|46.81006|-92.08174|9/14/2014 0:00|NE
1|10|31|1|46.81006|-92.08174|9/14/2014 0:00|N
```

-   **定义Extractor**

    写一个通用的Extractor，将分隔符作为参数传进来，可以处理所有类似格式的text文件。如下所示：

    ```
    /**
     * Text extractor that extract schematized records from formatted plain-text(csv, tsv etc.)
     **/
    public class TextExtractor extends Extractor {
      private InputStreamSet inputs;
      private String columnDelimiter;
      private DataAttributes attributes;
      private BufferedReader currentReader;
      private boolean firstRead = true;
      public TextExtractor() {
        // default to ",", this can be overwritten if a specific delimiter is provided (via DataAttributes)
        this.columnDelimiter = ",";
      }
      // no particular usage for execution context in this example
      @Override
      public void setup(ExecutionContext ctx, InputStreamSet inputs, DataAttributes attributes) {
        this.inputs = inputs; //  inputs 是一个 InputStreamSet，每次调用 next() 返回一个 InputStream，这个 InputStream 可以读取一个 OSS 文件的所有内容。
        this.attributes = attributes;
        // check if "delimiter" attribute is supplied via SQL query
        String columnDelimiter = this.attributes.getValueByKey("delimiter"); //delimiter 通过 DDL 语句传参。
        if ( columnDelimiter != null)
        {
          this.columnDelimiter = columnDelimiter;
        }
        // note: more properties can be inited from attributes if needed
      }
      @Override
      public Record extract() throws IOException {//extactor() 调用返回一条 Record，代表外部表中的一条记录。
        String line = readNextLine();
        if (line == null) {
          return null; // 返回 NULL 来表示这个表中已经没有记录可读。
        }
        return textLineToRecord(line); // textLineToRecord 将一行数据按照 delimiter 分割为多个列。
      }
      @Override
      public void close(){
        // no-op
      }
    }
    ```

    textLineToRecord将数据分割的完整实现请参见[此处](https://github.com/aliyun/aliyun-odps-java-sdk/blob/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text/TextExtractor.java)。

    **定义StorageHandler**

    StorageHandler作为External Table自定义逻辑的统一入口。

    ```
    package com.aliyun.odps.udf.example.text;
    public class TextStorageHandler extends OdpsStorageHandler {
      @Override
      public Class<? extends Extractor> getExtractorClass() {
        return TextExtractor.class;
      }
      @Override
      public Class<? extends Outputer> getOutputerClass() {
        return TextOutputer.class;
      }
    }
    ```

    **编译打包**

    将自定义代码编译打包，并上传到MaxCompute。

    ```
    add jar odps-udf-example.jar;
    ```

-   **创建External表**

    与使用内置Extractor相似，首先需要创建一张外部表，不同的是在指定外部表访问数据的时候，需要使用自定义的StorageHandler。

    创建外部表语句如下:

    ```
    CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_txt_external
    (
    vehicleId int,
    recordId int,
    patientId int,
    calls int,
    locationLatitute double,
    locationLongtitue double,
    recordTime string,
    direction string
    )
    STORED BY 'com.aliyun.odps.udf.example.text.TextStorageHandler' --STORED BY 指定自定义 StorageHandler 的类名。
      with SERDEPROPERTIES (
    'delimiter'='\\|',  --SERDEPROPERITES 可以指定参数，这些参数会通过 DataAttributes 传递到 Extractor 代码中。
    'odps.properties.rolearn'='acs:ram::xxxxxxxxxxxxx:role/aliyunodpsdefaultrole'
    )
    LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/SampleData/CustomTxt/AmbulanceData/'
    USING 'odps-udf-example.jar'; --同时需要指定类定义所在的jar包。
    ```

-   **查询外部表**

    执行如下SQL语句：

    ```
    select recordId, patientId, direction from ambulance_data_txt_external where patientId > 25;
    ```


## 自定义Extractor访问非文本文件数据 {#section_kqm_bhb_wdb .section}

在前面我们看到了通过内置与自定义的Extractor可以轻松处理存储在OSS上的CSV等文本数据。接下来以语音数据（wav格式文件）为例，为您介绍如何通过自定义的Extractor访问并处理OSS上的非文本文件。

**这里从最终执行的SQL开始，介绍以MaxCompute SQL为入口，处理存放在OSS上的语音文件的使用方法。**

创建外部表SQL如下：

```
CREATE EXTERNAL TABLE IF NOT EXISTS speech_sentence_snr_external
(
sentence_snr double,
id string
)
STORED BY 'com.aliyun.odps.udf.example.speech.SpeechStorageHandler'
WITH SERDEPROPERTIES (
    'mlfFileName'='sm_random_5_utterance.text.label' ,
    'speechSampleRateInKHz' = '16'
)
LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/dev/SpeechSentenceTest/'
USING 'odps-udf-example.jar,sm_random_5_utterance.text.label';
```

如上所示，同样需要创建外部表，然后通过外部表的Schema定义了希望通过外部表从语音文件中抽取出来的信息：

-   一个语音文件中的语句信噪比（SNR）：sentence\_snr。
-   对应语音文件的名字：id。

创建外部表后，通过标准的Select语句进行查询，则会触发Extractor运行计算。此处便可感受到，在读取处理OSS数据时，除了可以对文本文件做简单的反序列化处理，还可以通过自定义Extractor实现更复杂的数据处理抽取逻辑。比如：在此示例中，通过自定义的`com.aliyun.odps.udf.example.speech.SpeechStorageHandler` 中封装的Extractor，实现了对语音文件计算平均有效语句信噪比的功能，并将抽取出来的结构化数据直接进行SQL运算（WHERE sentence\_snr \> 10），最终返回所有信噪比大于10的语音文件以及对应的信噪比值。

在OSS地址`oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/dev/SpeechSentenceTest/`上，存储了原始的多个WAV格式的语音文件，MaxCompute框架将读取该地址上的所有文件，并在必要的时候进行文件级别的分片，自动将文件分配给多个计算节点处理。每个计算节点上的Extractor则负责处理通过InputStreamSet分配给该节点的文件集。具体的处理逻辑则与用户单机程序相仿，您不需关心分布计算中的种种细节，按照类单机方式实现其用户算法即可。

定制化的`SpeechSentenceSnrExtractor`主体逻辑，说明如下：

首先在`setup`接口中读取参数，进行初始化，并且导入语音处理模型（通过resource引入）：

```
public SpeechSentenceSnrExtractor(){
    this.utteranceLabels = new HashMap<String, UtteranceLabel>();
  }
  @Override
  public void setup(ExecutionContext ctx, InputStreamSet inputs, DataAttributes attributes){
    this.inputs = inputs;
    this.attributes = attributes;
    this.mlfFileName = this.attributes.getValueByKey(MLF_FILE_ATTRIBUTE_KEY);
    String sampleRateInKHzStr = this.attributes.getValueByKey(SPEECH_SAMPLE_RATE_KEY);
    this.sampleRateInKHz = Double.parseDouble(sampleRateInKHzStr);
    try {
      // read the speech model file from resource and load the model into memory
      BufferedInputStream inputStream = ctx.readResourceFileAsStream(mlfFileName);
      loadMlfLabelsFromResource(inputStream);
      inputStream.close();
    } catch (IOException e) {
      throw new RuntimeException("reading model from mlf failed with exception " + e.getMessage());
    }
  }

```

Extractor\(\)接口中，实现了对语音文件的具体读取和处理逻辑，对读取的数据根据语音模型进行信噪比的计算，并且将结果填充成\[snr, id\]格式的Record。

上述示例对实现进行了简化，同时也没有包括涉及语音处理的算法逻辑，具体实现请参见MaxCompute SDK在开源社区中提供的[样例代码](https://github.com/aliyun/aliyun-odps-java-sdk/blob/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/speech/SpeechSentenceSnrExtractor.java)。

```
@Override
  public Record extract() throws IOException {
    SourceInputStream inputStream = inputs.next();
    if (inputStream == null){
      return null;
    }
    // process one wav file to extract one output record [snr, id]
    String fileName = inputStream.getFileName();
    fileName = fileName.substring(fileName.lastIndexOf('/') + 1);
    logger.info("Processing wav file " + fileName);
    String id = fileName.substring(0, fileName.lastIndexOf('.'));
    // read speech file into memory buffer
    long fileSize = inputStream.getFileSize();
    byte[] buffer = new byte[(int)fileSize];
    int readSize = inputStream.readToEnd(buffer);
    inputStream.close();
    // compute the avg sentence snr
    double snr = computeSnr(id, buffer, readSize);
    // construct output record [snr, id]
    Column[] outputColumns = this.attributes.getRecordColumns();
    ArrayRecord record = new ArrayRecord(outputColumns);
    record.setDouble(0, snr);
    record.setString(1, id);
    return record;
  }
  private void loadMlfLabelsFromResource(BufferedInputStream fileInputStream)
          throws IOException {
    //  skipped here
  }
  // compute the snr of the speech sentence, assuming the input buffer contains the entire content of a wav file
  private double computeSnr(String id, byte[] buffer, int validBufferLen){
    // computing the snr value for the wav file (supplied as byte buffer array), skipped here
  }
```

执行查询，如下所示：

```
select sentence_snr, id
    from speech_sentence_snr_external
where sentence_snr > 10.0;
```

获得计算结果，如下所示：

```
--------------------------------------------------------------
| sentence_snr |                     id                      |
--------------------------------------------------------------
|   34.4703    |          J310209090013_H02_K03_042          |
--------------------------------------------------------------
|   31.3905    | tsh148_seg_2_3013_3_6_48_80bd359827e24dd7_0 |
--------------------------------------------------------------
|   35.4774    | tsh148_seg_3013_1_31_11_9d7c87aef9f3e559_0  |
--------------------------------------------------------------
|   16.0462    | tsh148_seg_3013_2_29_49_f4cb0990a6b4060c_0  |
--------------------------------------------------------------
|   14.5568    |   tsh_148_3013_5_13_47_3d5008d792408f81_0   |
--------------------------------------------------------------
```

综上所述，通过自定义Extractor，便可在SQL语句上分布式地处理多个OSS上的语音数据文件。同样的方法，也可以方便地利用MaxCompute的大规模计算能力，完成对图像，视频等各种类型非结构化数据的处理。

## 数据的分区 {#section_avv_rhb_wdb .section}

在前面的例子中，一个外部表关联的数据通过LOCATION上指定的OSS目录来实现，而在处理的时候，MaxCompute是读取目录下的所有数据，**包括子目录中的所有文件**。在数据量比较大，尤其是对于随着时间不断积累的数据目录，对全目录扫描可能带来不必要的I/O以及数据处理时间。 解决这个问题通常有两种做法：

-   直接的方法：用户对数据存放地址做好规划，考虑使用多个EXTERNAL TABLE来描述不同部分的数据，让每个EXTERNALTABLE的LOCATION指向数据的一个子集。
-   数据分区方法：EXTERNAL TABLE与内部表一样，**支持分区表的功能**，可以通过这个功能来对数据做系统化的管理。

本章节主要介绍EXTERNAL TABLE的分区功能。

-   **分区数据在OSS上的标准组织方式和路径格式**

    与MaxCompute内部表不同，对于存放在外部存储上\(如OSS\)上面的数据，MaxComput没有数据的管理权，因此如果需要使用分区表功能，在OSS上数据文件的存放路径必须符合一定的格式，路径格式如下：

    ```
    partitionKey1=value1\partitionKey2=value2\...
    ```

    **场景示例**

    将每天产生的LOG文件存放在OSS上，并需要通过MaxCompute进行数据处理，数据处理时需按照粒度为“天”来访问一部分数据。假设这些LOG文件为CSV格式且可以用内置extractor访问（复杂自定义格式用法也类似），那么**外部分区表**定义数据如下：

    ```
    CREATE EXTERNAL TABLE log_table_external (
        click STRING,
        ip STRING,
        url STRING,
      )
      PARTITIONED BY (
        year STRING,
        month STRING,
        day STRING
      )
      STORED BY 'com.aliyun.odps.CsvStorageHandler'
      WITH SERDEPROPERTIES (
     'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
    ) 
      LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/log_data/';
    ```

    如上建表语句，和前面的例子区别在于定义EXTERNAL TABLE时，通过PARTITIONED BY的语法指定该外部表为分区表，该例子是一个三层分区分区表，分区的key分别是year，month和day。

    为了让分区生效，在OSS上存储数据时需要遵循location的路径格式。如有效的路径存储layout：

    ```
    osscmd ls oss://oss-odps-test/log_data/
    2017-01-14 08:03:35 128MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=01/logfile
    2017-01-14 08:04:12 127MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=01/logfile.1
    2017-01-14 08:05:02 118MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=02/logfile
    2017-01-14 08:06:45 123MB Standard oss://oss-odps-test/log_data/year=2016/month=07/day=10/logfile
    2017-01-14 08:07:11 115MB Standard oss://oss-odps-test/log_data/year=2016/month=08/day=08/logfile
    ...
    ```

    **说明：** 因为数据是离线准备的，即通过osscmd或者其他OSS工具上载到OSS存储服务，所以数据路径格式也在上载时决定。

    通过ALTER TABLE ADD PARTITIONDDL语句，即可把这些分区信息引入MaxCompute。

    对应的DDL语句：

    ```
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '01')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '02')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '07', day = '10')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '08', day = '08')
    ...
    ```

    **说明：** 以上这些操作与标准的MaxCompute内部表操作一样，对[分区表](../cn.zh-CN/产品简介/基本概念/分区.md)概念不熟悉的可以参考文档。在数据准备好并且PARTITION信息引入MaxCompute之后，即可通过SQL语句对OSS外表数据的分区进行操作。

    此时分析数据时，可以指定指需分析某天的数据，如只想分析2016年6月1号当天，有多少不同的IP出现在LOG里面，可以通过如下语句实现：

    ```
    SELECT count(distinct(ip)) FROM log_table_external WHERE year = '2016' AND month = '06' AND day = '01';
    ```

    该语句对log\_table\_external这个外表对应的目录，将只访问`log_data/year=2016/month=06/day=01`子目录下的文件\(logfile和logfile.1\)，**不会对整个**log\_data/**目录作全量数据扫描，避免大量无用的I/O操作。**

    同样如果只希望对2016年下半年的数据做分析，则：

    ```
    SELECT count(distinct(ip)) FROM log_table_external 
    WHERE year = '2016' AND month > '06';
    ```

    只访问OSS上面存储的下半年的LOG数据。

-   **分区数据在OSS上的自定义路径**

    如果事先存在OSS上的历史数据，但是又不是根据`partitionKey1=value1\partitionKey2=value2\...`路径格式来组织存放，也需要通过MaxCompute的分区方式来进行访问计算时，MaxCompute也提供了通过自定义路径来引入partition的方法。

    假设OSS数据路径只有简单的分区值（而无分区key信息），也就是数据的layout为：

    ```
    osscmd ls oss://oss-odps-test/log_data_customized/
    2017-01-14 08:03:35 128MB Standard oss://oss-odps-test/log_data_customized/2016/06/01/logfile
    2017-01-14 08:04:12 127MB Standard oss://oss-odps-test/log_data_customized/2016/06/01/logfile.1
    2017-01-14 08:05:02 118MB Standard oss://oss-odps-test/log_data_customized/2016/06/02/logfile
    2017-01-14 08:06:45 123MB Standard oss://oss-odps-test/log_data_customized/2016/07/10/logfile
    2017-01-14 08:07:11 115MB Standard oss://oss-odps-test/log_data_customized/2016/08/08/logfile
    ...
    ```


外部表建表DDL可参看前面的示例，同样在建表语句里指定好分区key。

不同的子目录指定到不同的分区，可通过类似如下自定义分区路径的DDL语句实现。

```
ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '01')
LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/log_data_customized/2016/06/01/';
```

在ADD PARTITION的时候增加了LOCATION信息，从而实现自定义分区数据路径后，即使数据存放不符合推荐的`partitionKey1=value1\partitionKey2=value2\...`格式，也能正确的实现对子目录数据的分区访问了。

