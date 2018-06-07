# Sort示例 {#concept_ryq_qfl_vdb .concept}

## 测试准备 {#section_e3n_syg_vdb .section}

1.  准备好测试程序的 Jar 包，假设名字为 mapreduce-examples.jar，本地存放路径为data\\resources。
2.  准备好 Sort 的测试表和资源。
    -   创建测试表。

        ```
        create table ss_in(key bigint, value bigint);
        create table ss_out(key bigint, value bigint);
        ```

    -   添加测试资源。

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  使用 tunnel 导入数据。

    ```
    tunnel upload data ss_in;
    ```

    导入 ss\_in 表的数据文件 data 的内容，如下所示：

    ```
     2,1
     1,1
     3,1
    ```


## 测试步骤 {#section_rlv_bzg_vdb .section}

在 odpscmd 中执行 Sort，如下所示：

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.Sort ss_in ss_out;
```

## 预期结果 {#section_hzz_dzg_vdb .section}

作业成功结束后，输出表 ss\_out 中的内容，如下所示：

```
+------------+------------+
| key        | value      |
+------------+------------+
| 1          | 1          |
| 2          | 1          |
| 3          | 1          |
+------------+------------+
```

## 代码示例 {#section_jgb_gzg_vdb .section}

```
    package com.aliyun.odps.mapred.open.example;
    import java.io.IOException;
    import java.util.Date;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.data.TableInfo;
    import com.aliyun.odps.mapred.JobClient;
    import com.aliyun.odps.mapred.MapperBase;
    import com.aliyun.odps.mapred.TaskContext;
    import com.aliyun.odps.mapred.conf.JobConf;
    import com.aliyun.odps.mapred.example.lib.IdentityReducer;
    import com.aliyun.odps.mapred.utils.InputUtils;
    import com.aliyun.odps.mapred.utils.OutputUtils;
    import com.aliyun.odps.mapred.utils.SchemaUtils;
    /**
     * This is the trivial map/reduce program that does absolutely nothing other
     * than use the framework to fragment and sort the input values.
     *
     **/
    public class Sort {
      static int printUsage() {
        System.out.println("sort <input> <output>");
        return -1;
      }
      /**
       * Implements the identity function, mapping record's first two columns to
       * outputs.
       **/
      public static class IdentityMapper extends MapperBase {
        private Record key;
        private Record value;
        @Override
        public void setup(TaskContext context) throws IOException {
          key = context.createMapOutputKeyRecord();
          value = context.createMapOutputValueRecord();
        }
        @Override
        public void map(long recordNum, Record record, TaskContext context)
            throws IOException {
          key.set(new Object[] { (Long) record.get(0) });
          value.set(new Object[] { (Long) record.get(1) });
          context.write(key, value);
        }
      }
      /**
       * The main driver for sort program. Invoke this method to submit the
       * map/reduce job.
       *
       * @throws IOException
       *           When there is communication problems with the job tracker.
       **/
      public static void main(String[] args) throws Exception {
        JobConf jobConf = new JobConf();
        jobConf.setMapperClass(IdentityMapper.class);
        jobConf.setReducerClass(IdentityReducer.class);
        jobConf.setNumReduceTasks(1);
        jobConf.setMapOutputKeySchema(SchemaUtils.fromString("key:bigint"));
        jobConf.setMapOutputValueSchema(SchemaUtils.fromString("value:bigint"));
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), jobConf);
        OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), jobConf);
        Date startTime = new Date();
        System.out.println("Job started: " + startTime);
        JobClient.runJob(jobConf);
        Date end_time = new Date();
        System.out.println("Job ended: " + end_time);
        System.out.println("The job took "
            + (end_time.getTime() - startTime.getTime()) / 1000 + " seconds.");
      }
    }

```

