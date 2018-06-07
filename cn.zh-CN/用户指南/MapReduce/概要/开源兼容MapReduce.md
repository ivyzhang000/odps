# 开源兼容MapReduce {#concept_yc2_cyf_vdb .concept}

MaxCompute（原ODPS）有一套原生的MapReduce编程模型和接口，简单来说，这套接口的输入输出都是MaxCompute中的Table，处理的数据以Record为组织形式，它可以很好地描述Table中的数据处理过程。

但是与社区的Hadoop相比，编程接口差异较大。Hadoop用户如果要将原来的Hadoop MR作业迁移到MaxCompute的MR中执行，需要重写MR的代码，使用MaxCompute的接口进行编译和调试，运行正常后再打成一个Jar包才能放到MaxCompute的平台来运行。这个过程十分繁琐，需要耗费很多的开发和测试人力。如果能够完全不改或者少量地修改原来的Hadoop MapReduce代码，便可在MaxCompute平台上运行，将会比较理想。

现在MaxCompute平台提供了一个Hadoop MapReduce到MaxCompute MR的适配工具，已经在一定程度上实现了Hadoop MapReduce作业的二进制级别的兼容，即您可以在不改代码的情况下通过指定一些配置，便可将原来在Hadoop上运行的MapReduce Jar包拿过来直接运行在MaxCompute上。您可通过[此处](http://repo.aliyun.com/download/hadoop2openmr-1.0.jar)下载开发插件。目前该插件处于测试阶段，暂时还不能支持您自定义comparator和自定义key类型。

下文将以WordCount程序为例，为您介绍HadoopMR插件的基本使用方式。

**说明：** 

-   更多开源兼容性的介绍请参见[开源版本SDK兼容性](cn.zh-CN/用户指南/MapReduce/Java SDK/兼容版本SDK概述.md)。
-   更多Hadoop MapReduce SDK的介绍请参见[MapReduce官方文档](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html)。

## 下载HadoopMR插件 {#section_cb2_l1g_vdb .section}

请单击[此处](http://repo.aliyun.com/download/hadoop2openmr-1.0.jar)下载插件，包名为hadoop2openmr-1.0.jar。

**说明：** 这个Jar包中已经包含hadoop-2.7.2版本的相关依赖，在作业的Jar包中请不要携带Hadoop的依赖，避免版本冲突。

## 准备Jar包 {#section_byf_p1g_vdb .section}

编译导出WordCount的Jar包（wordcount\_test.jar），WordCount程序的源码如下所示：

```
package com.aliyun.odps.mapred.example.hadoop;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import java.io.IOException;
import java.util.StringTokenizer;
public class WordCount {
    public static class TokenizerMapper
        extends Mapper<Object, Text, Text, IntWritable>{
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }
    public static class IntSumReducer
        extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();
        public void reduce(Text key, Iterable<IntWritable> values,
            Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

```

## 准备测试数据 {#section_y1l_s1g_vdb .section}

1.  创建输入表和输出表。

    ```
    create table if not exists wc_in(line string);
    create table if not exists wc_out(key string, cnt bigint);
    ```

2.  通过Tunnel命令将数据导入输入表中。

    需要导入文本文件data.txt的数据内容如下：

    ```
    hello maxcompute
    hello mapreduce
    ```

    您可通过MaxCompute[客户端](../cn.zh-CN/工具及下载/客户端.md)的`Tunnel`命令将data.txt的数据导入wc\_in中，如下所示：

    ```
    tunnel upload data.txt wc_in;
    ```


## 准备表与HDFS文件路径的映射关系配置 {#section_b22_1bg_vdb .section}

配置文件命名为：wordcount-table-res.conf

```
{
  "file:/foo": {
    "resolver": {
      "resolver": "com.aliyun.odps.mapred.hadoop2openmr.resolver.TextFileResolver",
      "properties": {
          "text.resolver.columns.combine.enable": "true",
          "text.resolver.seperator": "\t"
      }
    },
    "tableInfos": [
      {
        "tblName": "wc_in",
        "partSpec": {},
        "label": "__default__"
      }
    ],
    "matchMode": "exact"
  },
  "file:/bar": {
    "resolver": {
      "resolver": "com.aliyun.odps.mapred.hadoop2openmr.resolver.BinaryFileResolver",
      "properties": {
          "binary.resolver.input.key.class" : "org.apache.hadoop.io.Text",
          "binary.resolver.input.value.class" : "org.apache.hadoop.io.LongWritable"
      }
    },
    "tableInfos": [
      {
        "tblName": "wc_out",
        "partSpec": {},
        "label": "__default__"
      }
    ],
    "matchMode": "fuzzy"
  }
}
```

配置项说明：

整个配置是一个JSON文件，描述HDFS上的文件与MaxCompute上的表之间的映射关系，一般要配置输入和输出两部分，一个HDFS路径对应一个resolver配置，tableInfos配置以及matchMode配置。

-   resolver：用于配置如何对待文件中的数据，目前有 com.aliyun.odps.mapred.hadoop2openmr.resolver.TextFileResolver和com.aliyun.odps.mapred.hadoop2openmr.resolver.BinaryFileResolver两个内置的resolver可以选用。除了指定好resolver的名字，还需要为相应的resolver配置一些properties指导它正确的进行数据解析。
    -   TextFileResolver：对于纯文本的数据，输入输出都会当成纯文本对待。当作为输入resolver配置时，需要配置的properties有：text.resolver.columns.combine.enable和text.resolver.seperator，当text.resolver.columns.combine.enable配置为true时，会把输入表的所有列按照text.resolver.seperator指定的分隔符组合成一个字符串作为输入。否则，会把输入表的前两列分别作为key，value。
    -   BinaryFileResolver：可以处理二进制的数据，自动将数据转换为MaxCompute可以支持的数据类型，如：Bigint，Bool，Double等。当作为输出resolver配置时，需要配置的properties有：binary.resolver.input.key.class和binary.resolver.input.value.class，分别代表中间结果的key和value类型。
-   tableInfos：您配置HDFS对应的MaxCompute表，目前只支持配置表的名字tblName，而partSpec和label请保持和示例一致。
-   matchMode：路径的匹配模式，可选项为exact和fuzzy，分别代表精确匹配和模糊匹配，如果设置为fuzzy，则可以通过正则来匹配HDFS的输入路径。

## 作业提交 {#section_okw_gbg_vdb .section}

使用MaxCompute命令行工具odpscmd提交作业。MaxCompute命令行工具的安装和配置方法请参见[客户端用户手册](../cn.zh-CN/工具及下载/客户端.md)。在odpscmd运行如下命令：

```
jar -DODPS_HADOOPMR_TABLE_RES_CONF=./wordcount-table-res.conf -classpath hadoop2openmr-1.0.jar,wordcount_test.jar com.aliyun.odps.mapred.example.hadoop.WordCount /foo /bar;
```

**说明：** 

-   wordcount-table-res.conf是配置了/foo /bar的映射。
-   wordcount\_test.jar包是您的Hadoop MapReduce的jar包。
-   com.aliyun.odps.mapred.example.hadoop.WordCount是您要运行的作业类名。
-   /foo /bar是指在HDFS上的路径，在JSON配置文件中映射成了wc\_in和wc\_out。
-   配置好映射后，您需要通过datax或DataWorks数据集成功能，手动导入Hadoop HDFS的输入文件到wc\_in进行MR计算，并手动导出结果数据wc\_out到您的HDFS输出目录/bar。
-   此处假设已经将hadoop2openmr-1.0.jar、wordcount\_test.jar和wordcount-table-res.conf放置到odpscmd的当前目录，否则在指定配置和-classpath的路径时需要做相应的修改。

运行过程如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12015/1957_zh-CN.jpg)

当作业运行完成后，便可查看结果表wc\_out的内容，验证作业是否成功结束，结果是否符合预期。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12015/1959_zh-CN.jpg)

