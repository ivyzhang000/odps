# WordCount示例 {#concept_rtv_dhg_vdb .concept}

## 测试准备 {#section_bjk_nxg_vdb .section}

1.  准备好测试程序的 Jar 包，假设名字为 mapreduce-examples.jar，本地存放路径为 data\\resources。
    -   创建测试表。

        ```
        create table wc_in (key string, value string);
        create table wc_out(key string, cnt bigint);
        ```

    -   添加测试资源。

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

2.  准备好 WordCount 测试表和资源。
3.  使用 tunnel 导入数据。

    ```
    tunnel upload data wc_in;
    ```

    导入 wc\_in 表的数据文件 data 的内容，如下所示：

    ```
    hello,odps
    ```


## 测试步骤 {#section_bhf_1yg_vdb .section}

在 odpscmd 中执行 WordCount，如下所示：

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.WordCount wc_in wc_out
```

## 预期结果 {#section_nvs_dyg_vdb .section}

作业成功结束后，输出表 wc\_out 中的内容，如下所示：

```
+------------+------------+
| key        | cnt        |
+------------+------------+
| hello      | 1          |
| odps       | 1          |
+------------+------------+
```

## 代码示例 {#section_ykc_gyg_vdb .section}

```
    package com.aliyun.odps.mapred.open.example;
    import java.io.IOException;
    import java.util.Iterator;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.data.TableInfo;
    import com.aliyun.odps.mapred.JobClient;
    import com.aliyun.odps.mapred.MapperBase;
    import com.aliyun.odps.mapred.ReducerBase;
    import com.aliyun.odps.mapred.TaskContext;
    import com.aliyun.odps.mapred.conf.JobConf;
    import com.aliyun.odps.mapred.utils.InputUtils;
    import com.aliyun.odps.mapred.utils.OutputUtils;
    import com.aliyun.odps.mapred.utils.SchemaUtils;
    public class WordCount {
      public static class TokenizerMapper extends MapperBase {
        private Record word;
        private Record one;
        @Override
        public void setup(TaskContext context) throws IOException {
          word = context.createMapOutputKeyRecord();
          one = context.createMapOutputValueRecord();
          one.set(new Object[] { 1L });
          System.out.println("TaskID:" + context.getTaskID().toString());
        }
        @Override
        public void map(long recordNum, Record record, TaskContext context)
            throws IOException {
          for (int i = 0; i < record.getColumnCount(); i++) {
            word.set(new Object[] { record.get(i).toString() });
            context.write(word, one);
          }
        }
      }
      /**
       * A combiner class that combines map output by sum them.
       **/
      public static class SumCombiner extends ReducerBase {
        private Record count;
        @Override
        public void setup(TaskContext context) throws IOException {
          count = context.createMapOutputValueRecord();
        }
        @Override
        public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
          long c = 0;
          while (values.hasNext()) {
            Record val = values.next();
            c += (Long) val.get(0);
          }
          count.set(0, c);
          context.write(key, count);
        }
      }
      /**
       * A reducer class that just emits the sum of the input values.
       **/
      public static class SumReducer extends ReducerBase {
        private Record result = null;
        @Override
        public void setup(TaskContext context) throws IOException {
          result = context.createOutputRecord();
        }
        @Override
        public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
          long count = 0;
          while (values.hasNext()) {
            Record val = values.next();
            count += (Long) val.get(0);
          }
          result.set(0, key.get(0));
          result.set(1, count);
          context.write(result);
        }
      }
      public static void main(String[] args) throws Exception {
        if (args.length != 2) {
          System.err.println("Usage: WordCount <in_table> <out_table>");
          System.exit(2);
        }
        JobConf job = new JobConf();
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(SumCombiner.class);
        job.setReducerClass(SumReducer.class);
        job.setMapOutputKeySchema(SchemaUtils.fromString("word:string"));
        job.setMapOutputValueSchema(SchemaUtils.fromString("count:bigint"));
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
        OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
        JobClient.runJob(job);
      }
    }

```

