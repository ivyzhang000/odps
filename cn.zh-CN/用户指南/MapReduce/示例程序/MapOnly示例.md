# MapOnly示例 {#concept_wz1_4yg_vdb .concept}

对于 MapOnly 的作业，Map 直接将 < Key, Value \> 信息输出到 MaxCompute 的表中，您只需要指定输出表即可，不需指定 Map 输出的 Key/Value 元信息。

## 测试准备 {#section_e3n_syg_vdb .section}

1.  准备好测试程序的 Jar 包，假设名字为 mapreduce-examples.jar，本地存放路径为 data\\resources。
2.  准备好 MapOnly 的测试表和资源。
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
     hello,odps
    ```


## 测试步骤 {#section_rlv_bzg_vdb .section}

在 odpscmd 中执行 MapOnly，如下所示：

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.MapOnly wc_in wc_out map
```

## 预期结果 {#section_hzz_dzg_vdb .section}

作业成功结束后，输出表 wc\_out 中的内容，如下所示：

```
+------------+------------+
| key        | cnt        |
+------------+------------+
| hello      | 1          |
| hello      | 1          |
+------------+------------+
```

## 代码示例 {#section_jgb_gzg_vdb .section}

```
    package com.aliyun.odps.mapred.open.example;
    import java.io.IOException;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.mapred.JobClient;
    import com.aliyun.odps.mapred.MapperBase;
    import com.aliyun.odps.mapred.conf.JobConf;
    import com.aliyun.odps.mapred.utils.SchemaUtils;
    import com.aliyun.odps.mapred.utils.InputUtils;
    import com.aliyun.odps.mapred.utils.OutputUtils;
    import com.aliyun.odps.data.TableInfo;
    public class MapOnly {
      public static class MapperClass extends MapperBase {
        @Override
        public void setup(TaskContext context) throws IOException {
          boolean is = context.getJobConf().getBoolean("option.mapper.setup", false);
          if (is) {
            Record result = context.createOutputRecord();
            result.set(0, "setup");
            result.set(1, 1L);
            context.write(result);
          }
        }
        @Override
        public void map(long key, Record record, TaskContext context) throws IOException {
          boolean is = context.getJobConf().getBoolean("option.mapper.map", false);
          if (is) {
            Record result = context.createOutputRecord();
            result.set(0, record.get(0));
            result.set(1, 1L);
            context.write(result);
          }
        }
        @Override
        public void cleanup(TaskContext context) throws IOException {
          boolean is = context.getJobConf().getBoolean("option.mapper.cleanup", false);
          if (is) {
            Record result = context.createOutputRecord();
            result.set(0, "cleanup");
            result.set(1, 1L);
            context.write(result);
          }
        }
      }
      public static void main(String[] args) throws Exception {
        if (args.length != 2 && args.length != 3) {
          System.err.println("Usage: OnlyMapper <in_table> <out_table> [setup|map|cleanup]");
          System.exit(2);
        }
        JobConf job = new JobConf();
        job.setMapperClass(MapperClass.class);
        job.setNumReduceTasks(0);
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
        OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
        if (args.length == 3) {
          String options = new String(args[2]);
          if (options.contains("setup")) {
            job.setBoolean("option.mapper.setup", true);
          }
          if (options.contains("map")) {
            job.setBoolean("option.mapper.map", true);
          }
          if (options.contains("cleanup")) {
            job.setBoolean("option.mapper.cleanup", true);
          }
        }
        JobClient.runJob(job);
      }
    }

```

