# Pipeline示例 {#concept_kss_3zf_vdb .concept}

## 测试准备 {#section_e3n_syg_vdb .section}

1.  准备好测试程序的 Jar 包，假设名字为 mapreduce-examples.jar，本地存放路径为data\\resources。
2.  准备好 Pipeline 的测试表和资源。
    -   创建测试表。

        ```
        create table wc_in (key string, value string);
        create table wc_out(key string, cnt bigint);
        ```

    -   添加测试资源。

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  使用 tunnel 导入数据。

    ```
    tunnel upload data wc_in;
    ```

    导入 wc\_in 表的数据文件 data 的内容，如下所示：

    ```
    hello,odps
    ```


## 测试步骤 {#section_rlv_bzg_vdb .section}

在 odpscmd 中执行 WordCountPipeline，如下所示：

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.WordCountPipeline wc_in wc_out;
```

## 预期结果 {#section_hzz_dzg_vdb .section}

作业成功结束后，输出表 wc\_out 中的内容，如下所示：

```
+------------+------------+
| key        | cnt        |
+------------+------------+
| hello      | 1          |
| odps       | 1          |
+------------+------------+
```

## 代码示例 {#section_jgb_gzg_vdb .section}

```
    package com.aliyun.odps.mapred.open.example;
    import java.io.IOException;
    import java.util.Iterator;
    import com.aliyun.odps.Column;
    import com.aliyun.odps.OdpsException;
    import com.aliyun.odps.OdpsType;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.data.TableInfo;
    import com.aliyun.odps.mapred.Job;
    import com.aliyun.odps.mapred.MapperBase;
    import com.aliyun.odps.mapred.ReducerBase;
    import com.aliyun.odps.pipeline.Pipeline;
    public class WordCountPipelineTest {
      public static class TokenizerMapper extends MapperBase {
        Record word;
        Record one;
        @Override
        public void setup(TaskContext context) throws IOException {
          word = context.createMapOutputKeyRecord();
          one = context.createMapOutputValueRecord();
          one.setBigint(0, 1L);
        }
        @Override
        public void map(long recordNum, Record record, TaskContext context)
            throws IOException {
          for (int i = 0; i < record.getColumnCount(); i++) {
            String[] words = record.get(i).toString().split("\\s+");
            for (String w : words) {
              word.setString(0, w);
              context.write(word, one);
            }
          }
        }
      }
      public static class SumReducer extends ReducerBase {
        private Record value;
        @Override
        public void setup(TaskContext context) throws IOException {
          value = context.createOutputValueRecord();
        }
        @Override
        public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
          long count = 0;
          while (values.hasNext()) {
            Record val = values.next();
            count += (Long) val.get(0);
          }
          value.set(0, count);
          context.write(key, value);
        }
      }
      public static class IdentityReducer extends ReducerBase {
        private Record result;
        @Override
        public void setup(TaskContext context) throws IOException {
          result = context.createOutputRecord();
        }
        @Override
        public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
          while (values.hasNext()) {
            result.set(0, key.get(0));
            result.set(1, values.next().get(0));
            context.write(result);
          }
        }
      }
      public static void main(String[] args) throws OdpsException {
        if (args.length != 2) {
          System.err.println("Usage: WordCountPipeline <in_table> <out_table>");
          System.exit(2);
        }
        Job job = new Job();
        /***
         * 构造Pipeline的过程中，如果不指定Mapper的OutputKeySortColumns，PartitionColumns，OutputGroupingColumns，
         * 框架会默认使用其OutputKey作为此三者的默认配置
         ***/
        Pipeline pipeline = Pipeline.builder()
            .addMapper(TokenizerMapper.class)
            .setOutputKeySchema(
                    new Column[] { new Column("word", OdpsType.STRING) })
            .setOutputValueSchema(
                    new Column[] { new Column("count", OdpsType.BIGINT) })
            .setOutputKeySortColumns(new String[] { "word" })
            .setPartitionColumns(new String[] { "word" })
            .setOutputGroupingColumns(new String[] { "word" })
            .addReducer(SumReducer.class)
            .setOutputKeySchema(
                    new Column[] { new Column("word", OdpsType.STRING) })
            .setOutputValueSchema(
                    new Column[] { new Column("count", OdpsType.BIGINT)})
            .addReducer(IdentityReducer.class).createPipeline();
        job.setPipeline(pipeline);
        job.addInput(TableInfo.builder().tableName(args[0]).build());
        job.addOutput(TableInfo.builder().tableName(args[1]).build());
        job.submit();
        job.waitForCompletion();
        System.exit(job.isSuccessful() == true ? 0 : 1);
      }
    }

```

