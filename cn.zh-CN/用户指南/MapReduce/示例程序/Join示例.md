# Join示例 {#concept_awd_xdh_vdb .concept}

MaxCompute MapReduce 框架自身并不支持 Join 逻辑，但您可以在自己的 Map/Reduce 函数中实现数据的 Join，当然这需要您做一些额外的工作。

假设需要 Join 两张表 mr\_Join\_src1\(key bigint, value string\) 和 mr\_Join\_src2\(key bigint, value string\)，输出表是 mr\_Join\_out\(key bigint, value1 string, value2 string\)，其中 value1 是 mr\_Join\_src1 的 value 值，value2 是 mr\_Join\_src2 的 value 值。

## 测试准备 {#section_e3n_syg_vdb .section}

1.  准备好测试程序的 Jar 包，假设名字为 mapreduce-examples.jar，本地存放路径为data\\resources。
2.  准备好 Join 的测试表和资源。
    -   创建测试表。

        ```
        create table mr_Join_src1(key bigint, value string);
        create table mr_Join_src2(key bigint, value string);
        create table mr_Join_out(key bigint, value1 string, value2 string);
        ```

    -   添加测试资源。

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  使用 tunnel 导入数据。

    ```
    tunnel upload data1 mr_Join_src1;
    tunnel upload data2 mr_Join_src2;
    ```

    导入 mr\_Join\_src1 数据的内容，如下所示：

    ```
     1,hello
     2,odps
    ```

    导入 mr\_Join\_src2 数据的内容，如下所示：

    ```
    1,odps
    3,hello
    4,odps
    ```


## 测试步骤 {#section_rlv_bzg_vdb .section}

在 odpscmd 中执行 Join，如下所示：

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.Join mr_Join_src1 mr_Join_src2 mr_Join_out;
```

## 预期结果 {#section_hzz_dzg_vdb .section}

作业成功结束后，输出表 mr\_Join\_out 中的内容，如下所示：

```
+------------+------------+------------+
| key        | value1     | value2     |
+------------+------------+------------+
|  1         | hello      |  odps      | 
+------------+------------+------------+
```

## 代码示例 {#section_jgb_gzg_vdb .section}

```
    package com.aliyun.odps.mapred.open.example;
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.Iterator;
    import java.util.List;
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.data.TableInfo;
    import com.aliyun.odps.mapred.JobClient;
    import com.aliyun.odps.mapred.MapperBase;
    import com.aliyun.odps.mapred.ReducerBase;
    import com.aliyun.odps.mapred.conf.JobConf;
    import com.aliyun.odps.mapred.utils.InputUtils;
    import com.aliyun.odps.mapred.utils.OutputUtils;
    import com.aliyun.odps.mapred.utils.SchemaUtils;
    /**
     * Join, mr_Join_src1/mr_Join_src2(key bigint, value string), mr_Join_out(key
     * bigint, value1 string, value2 string)
     * 
     */
    public class Join {
      public static final Log LOG = LogFactory.getLog(Join.class);
      public static class JoinMapper extends MapperBase {
        private Record mapkey;
        private Record mapvalue;
        private long tag;
        @Override
        public void setup(TaskContext context) throws IOException {
          mapkey = context.createMapOutputKeyRecord();
          mapvalue = context.createMapOutputValueRecord();
          tag = context.getInputTableInfo().getLabel().equals("left") ? 0 : 1;
        }
        @Override
        public void map(long key, Record record, TaskContext context)
            throws IOException {
          mapkey.set(0, record.get(0));
          mapkey.set(1, tag);
          for (int i = 1; i < record.getColumnCount(); i++) {
            mapvalue.set(i - 1, record.get(i));
          }
          context.write(mapkey, mapvalue);
        }
      }
      public static class JoinReducer extends ReducerBase {
        private Record result = null;
        @Override
        public void setup(TaskContext context) throws IOException {
          result = context.createOutputRecord();
        }
        @Override
        public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
          long k = key.getBigint(0);
          List<Object[]> leftValues = new ArrayList<Object[]>();
          while (values.hasNext()) {
            Record value = values.next();
            long tag = (Long) key.get(1);
            if (tag == 0) {
              leftValues.add(value.toArray().clone());
            } else {
              for (Object[] leftValue : leftValues) {
                int index = 0;
                result.set(index++, k);
                for (int i = 0; i < leftValue.length; i++) {
                  result.set(index++, leftValue[i]);
                }
                for (int i = 0; i < value.getColumnCount(); i++) {
                  result.set(index++, value.get(i));
                }
                context.write(result);
              }
            }
          }
        }
      }
      public static void main(String[] args) throws Exception {
        if (args.length != 3) {
          System.err.println("Usage: Join <input table1> <input table2> <out>");
          System.exit(2);
        }
        JobConf job = new JobConf();
        job.setMapperClass(JoinMapper.class);
        job.setReducerClass(JoinReducer.class);
        job.setMapOutputKeySchema(SchemaUtils.fromString("key:bigint,tag:bigint"));
        job.setMapOutputValueSchema(SchemaUtils.fromString("value:string"));
        job.setPartitionColumns(new String[]{"key"});
        job.setOutputKeySortColumns(new String[]{"key", "tag"});
        job.setOutputGroupingColumns(new String[]{"key"});
        job.setNumReduceTasks(1);
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).label("left").build(), job);
        InputUtils.addTable(TableInfo.builder().tableName(args[1]).label("right").build(), job);
        OutputUtils.addTable(TableInfo.builder().tableName(args[2]).build(), job);
        JobClient.runJob(job);
      }
    }

```

