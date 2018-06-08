# 输出到OSS的非结构化数据 {#concept_wsw_nnb_wdb .concept}

[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)为您介绍了MaxCompute如何通过外部表的关联进行访问并处理存储在OSS的非结构化数据，实际上MaxCompute的非结构化框架也支持通过insert方式将MaxCompute的数据直接输出到OSS，MaxCompute也是通过外部表关联OSS，进行数据输出。

输出数据到OSS通常是两种情况：

-   MaxCompute内部表输出到关联OSS的外部表。
-   MaxCompute处理外部表后结果直接输出到关联OSS的外部表。

与访问OSS数据一样，MaxCompute支持通过内置StorageHandler 和自定义StorageHandler进行输出。

## 通过内置StorageHandler输出到OSS {#section_xcn_snb_wdb .section}

使用MaxCompute内置的StorageHandler ，可以非常方便的按照约定格式输出数据到OSS进行存储。我们只需要创建一个外部表，指明内置的StorageHandler，就能以这张表为关联，相关逻辑由系统实现。

目前MaxCompute支持2个内置StorageHandler：

-   com.aliyun.odps.CsvStorageHandler ，定义如何读写csv格式数据，数据格式约定：英文逗号`,`为列分隔符，换行符为`\n`。
-   com.aliyun.odps.TsvStorageHandler，定义如何读写csv格式数据，数据格式约定：`\t`为列分隔符，换行符为`\n`。

-   **创建EXTERNAL TABLE**

    ```
    CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
    (<column schemas>)
    [PARTITIONED BY (partition column schemas)]
    STORED BY '<StorageHandler>'  
    [WITH SERDEPROPERTIES (  'odps.properties.rolearn'='${roleran}') ]
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
    
    ```

    -   STORED BY，如果需求输出到OSS上的数据文件是TSV文件，则用内置`com.aliyun.odps.TsvStorageHandler`；如果需求输出到OSS上的数据文件是CSV文件，则用内置`com.aliyun.odps.CsvStorageHandler`。
    -   WITH SERDEPROPERTIES，当关联OSS权限使用“STS模式授权”的“自定义授权”时，需要该参数指定’odps.properties.rolearn’属性，属性值为RAM 中具体使用的自定义role的Arn的信息。

        **说明：** STS模式授权可参看《[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)》中的对应内容。

    -   LOCATION，指定对应OSS存储的文件路径。若WITH SERDEPROPERTIES中不设置’odps.properties.rolearn’属性，且授权方式是采用明文AK，则LOCATION为

        ```
        LOCATION
            'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
        ```

-   **通过对External Table的 INSERT 操作实现数据输出到OSS**

    通过External Table关联上OSS存储路径后，可以对External Table做标准的SQL INSERT OVERWRITE/INSERT INTO操作既可将数据输出到OSS。

    ```
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];
    
    ```

    -   from\_tablename：可以是内部表，也可以是外部表（包括关联的OSS或OTS的外部表）。

    -   INSERT将按照外部表‘STORED BY’指定 ‘StorageHandler’的格式（即TSV或CSV）写到OSS上。

    INSERT 操作成功完成后，就可以看到OSS上的对应LOCATION产生了一系列文件。

    如：external table 对应的location是oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/tsv\_output\_folder/则，在OSS对应路径中可以看到生成一系列文件：

    ```
    osscmd ls oss://oss-odps-test/tsv_output_folder/
    2017-01-14 06:48:27 39.00B Standard oss://oss-odps-test/tsv_output_folder/.odps/.meta
    2017-01-14 06:48:12 4.80MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_0_0-0.tsv
    2017-01-14 06:48:05 4.78MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_1_0-0.tsv
    2017-01-14 06:47:48 4.79MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_2_0-0.tsv
    ...
    ```

    可以看到，通过前面LOCATION指定的oss-odps-test这个OSS bucket下的tsv\_output\_folder文件夹下产生了一个’.odps’文件夹，这其中将有一些’.tsv’文件，以及一个’.meta’文件。 类似的文件结构是MaxCompute往OSS上输出所特有的：

    -   通过MaxCompute对一个OSS地址，使用INSERT INTO/OVERWRITE 外部表来做输出操作，所有的数据将在指定的LOCATION下的’.odps’文件夹产生。
    -   ‘.odps’文件夹中的’.meta’文件为MaxCompute额外写出的宏数据文件，其中用于记录当前文件夹中有效的数据。 正常情况下，如果INSERT操作成功完成的话，可以认为当前文件夹的所有数据均是有效数据。 只有在有作业失败的情况下需要对这个宏数据进行解析。 即使是在作业中途失败或被kill的情况下，对于INSERT OVERWRITE操作，再跑一次成功即可。
    -   如果是分区表，将在tsv\_output\_folder文件夹下根据insert语句指定的分区值生成对应的分区子目录然后分区子目录里才是’.odps’文件夹。如`test/tsv_output_folder/一级分区名=分区值/n级分区名=分区值/.odps/20170113224724561g9m6csz7/M1_2_0-0.tsv`。
    对于MaxCompute内置的TSV/CSV StorageHandler 处理来说，产生的文件数目与对应SQL stage的并发度是相同。

    若`INSER OVERWITE ... SELECT ... FROM ...;`的操作在源数据表\(from\_tablename\) 上分配了1000个mapper，那么最后将产生了1000个TSV/CSV文件。


## 通过自定义StorageHandler输出到OSS {#section_tbq_44b_wdb .section}

除了使用内置的StorageHandler来实现在OSS上输出TSV/CSV常见文本格式，MaxCompute非结构化框架提供了通用的SDK，支持对外输出自定义数据格式文件。

与内置StorageHandler一样需要先“创建EXTERNAL TABLE”，再“通过对External Table的 INSERT 操作实现数据输出到OSS”。不同点在于创建外部表时，STORED BY是需要指定自定义的StorageHandler。

**说明：** MaxCompute非结构化框架通过StorageHandler这个接口来描述对各种数据存储格式的处理。 具体来说，StorageHandler作为一个wrapper class, 让您指定自定义的Exatractor\(用于数据的读入，解析，处理等\) 以及Outputer\(用于数据的处理和输出等\)。 自定义的StorageHandler 应该继承 OdpsStorageHandler，实现getExtractorClass以及getOutputerClass 两个接口。

接下来我们用[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)中“自定义 Extractor 访问 OSS”的‘TextStorageHandler ’例子来介绍MaxCompute如何通过自定义StorageHandler 将数据输出到OSS的txt文件，且以‘|’为列分隔符，以‘\\n’为换行符。

**说明：** [MaxCompute Studio](../cn.zh-CN/工具及下载/MaxCompute Studio/认识 Studio.md)配置好[MaxCompute Java Module](../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/创建 MaxCompute Java Module.md)后，可以在examples中看到对应的示例代码。或者点击[此处](https://github.com/aliyun/aliyun-odps-java-sdk/tree/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text)也看到完整代码。

-   **定义Outputer**

    输出逻辑都必须实现Outputer接口：

    ```
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.io.OutputStreamSet;
    import com.aliyun.odps.io.SinkOutputStream;
    import com.aliyun.odps.udf.DataAttributes;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.Outputer;
    import java.io.IOException;
    public class TextOutputer extends Outputer {
        private SinkOutputStream outputStream;
        private DataAttributes attributes;
        private String delimiter;
        public TextOutputer (){
            // default delimiter, this can be overwritten if a delimiter is provided through the attributes.
            this.delimiter = "|";
        }
        @Override
        public void output(Record record) throws IOException {
            this.outputStream.write(recordToString(record).getBytes());
        }
        // no particular usage of execution context in this example
        @Override
        public void setup(ExecutionContext ctx, OutputStreamSet outputStreamSet, DataAttributes attributes) throws IOException {
            this.outputStream = outputStreamSet.next();
            this.attributes = attributes;
        }
        @Override
        public void close() {
            // no-op
        }
        private String recordToString(Record record){
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < record.getColumnCount(); i++)
            {
                if (null == record.get(i)){
                    sb.append("NULL");
                }
                else{
                    sb.append(record.get(i).toString());
                }
                if (i != record.getColumnCount() - 1){
                    sb.append(this.delimiter);
                }
            }
            sb.append("\n");
            return sb.toString();
        }
    }
    ```

    Outputer接口有三个：setup, output和close, 这和Extractor的setup, extract和close三个接口基本上对称。 其中setup\(\)和close\(\)在一个outputer中只会调用一次。 可以在setup里面做初始化准备工作，另外通常需要把setup\(\)传递进来的这三个参数保存成ouputerd的class variable, 方便之后output\(\)或者close\(\)接口中使用。 而close\(\)这个接口用于代码的扫尾工作。

    通常情况下大部分的数据处理发生在output\(Record\)这个接口内。 MaxCompute系统根据当前outputer分配处理的每个输入Record调用一次 output\(Record\)。 假设在一个output\(Record\) 调用返回的时候，代码已经消费完这个Record, 因此在当前output\(Record\)返回后，系统会将Record所使用的内存作它用，所以当Record中的信息在跨多个output\(\)函数调用被使用时，需要调用当前处理的record.clone\(\)方法，将当前record保存下来。

-   **定义Extractor**

    Exatractor用于数据的读入，解析，处理等，如果输出的表最终不需要再通过MaxCompute进行读取等，可以无需定义。

    ```
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.Column;
    import com.aliyun.odps.data.ArrayRecord;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.io.InputStreamSet;
    import com.aliyun.odps.udf.DataAttributes;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.Extractor;
    import java.io.BufferedReader;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
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
            this.inputs = inputs;
            this.attributes = attributes;
            // check if "delimiter" attribute is supplied via SQL query
            String columnDelimiter = this.attributes.getValueByKey("delimiter");
            if ( columnDelimiter != null)
            {
                this.columnDelimiter = columnDelimiter;
            }
            System.out.println("TextExtractor using delimiter [" + this.columnDelimiter + "].");
            // note: more properties can be inited from attributes if needed
        }
        @Override
        public Record extract() throws IOException {
            String line = readNextLine();
            if (line == null) {
                return null;
            }
            return textLineToRecord(line);
        }
        @Override
        public void close(){
            // no-op
        }
        private Record textLineToRecord(String line) throws IllegalArgumentException
        {
            Column[] outputColumns = this.attributes.getRecordColumns();
            ArrayRecord record = new ArrayRecord(outputColumns);
            if (this.attributes.getRecordColumns().length != 0){
                // string copies are needed, not the most efficient one, but suffice as an example here
                String[] parts = line.split(columnDelimiter);
                int[] outputIndexes = this.attributes.getNeededIndexes();
                if (outputIndexes == null){
                    throw new IllegalArgumentException("No outputIndexes supplied.");
                }
                if (outputIndexes.length != outputColumns.length){
                    throw new IllegalArgumentException("Mismatched output schema: Expecting "
                            + outputColumns.length + " columns but get " + parts.length);
                }
                int index = 0;
                for(int i = 0; i < parts.length; i++){
                    // only parse data in columns indexed by output indexes
                    if (index < outputIndexes.length && i == outputIndexes[index]){
                        switch (outputColumns[index].getType()) {
                            case STRING:
                                record.setString(index, parts[i]);
                                break;
                            case BIGINT:
                                record.setBigint(index, Long.parseLong(parts[i]));
                                break;
                            case BOOLEAN:
                                record.setBoolean(index, Boolean.parseBoolean(parts[i]));
                                break;
                            case DOUBLE:
                                record.setDouble(index, Double.parseDouble(parts[i]));
                                break;
                            case DATETIME:
                            case DECIMAL:
                            case ARRAY:
                            case MAP:
                            default:
                                throw new IllegalArgumentException("Type " + outputColumns[index].getType() + " not supported for now.");
                        }
                        index++;
                    }
                }
            }
            return record;
        }
        /**
         * Read next line from underlying input streams.
         * @return The next line as String object. If all of the contents of input
         * streams has been read, return null.
         */
        private String readNextLine() throws IOException {
            if (firstRead) {
                firstRead = false;
                // the first read, initialize things
                currentReader = moveToNextStream();
                if (currentReader == null) {
                    // empty input stream set
                    return null;
                }
            }
            while (currentReader != null) {
                String line = currentReader.readLine();
                if (line != null) {
                    return line;
                }
                currentReader = moveToNextStream();
            }
            return null;
        }
        private BufferedReader moveToNextStream() throws IOException {
            InputStream stream = inputs.next();
            if (stream == null) {
                return null;
            } else {
                return new BufferedReader(new InputStreamReader(stream));
            }
        }
    }
    ```

    详情请参见[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)文档。

-   **定义StorageHandler**

    ```
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.udf.Extractor;
    import com.aliyun.odps.udf.OdpsStorageHandler;
    import com.aliyun.odps.udf.Outputer;
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

    若表无需读取可不用指定Extractor接口。

-   **编译打包**

    将自定义代码编译打包，并作为jar 资源上传到MaxCompute。如打的jar包名为‘odps-TextStorageHandler.jar’,则上传为MaxCompute Resource如下：

    ```
    add jar odps-TextStorageHandler.jar;
    ```

-   **创建external表**

    与使用内置StorageHandler类似，需要建立一个外部表，不同的是这次需要指定数据输出到外部表时候，使用自定义的StorageHandler。

    ```
    CREATE EXTERNAL TABLE IF NOT EXISTS output_data_txt_external
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
    STORED BY 'com.aliyun.odps.examples.unstructured.text.TextStorageHandler' 
    WITH SERDEPROPERTIES(
        'delimiter'='|'
        [,'odps.properties.rolearn'='${roleran}'])
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/'
    USING 'odps-TextStorageHandler.jar';
    ```

    **说明：** 若需用‘odps.properties.rolearn’属性，详情请参见[访问OSS非结构化数据](cn.zh-CN/用户指南/处理非结构化数据/访问OSS非结构化数据.md)的STS模式授权的**自定义授权**。若不用，可以参考**一键授权**或者在LOCATION上用明文AK。

-   **通过对External Table的INSERT操作实现数据输出到OSS**

    通过自定义StorageHandler 创建External Table关联上OSS存储路径后，可以对External Table做标准的SQL INSERT OVERWRITE/INSERT INTO操作既可将数据输出到OSS,方式与内置StorageHandler一样：

    ```
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];
    
    ```

    insert操作执行成功后，与内置StorageHandler一样，可以在OSS对应LOCATION路径看到生成一系列文件于’.odps’文件夹中。


