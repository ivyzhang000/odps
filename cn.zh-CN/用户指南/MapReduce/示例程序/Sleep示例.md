# Sleep示例 {#concept_zjq_s2h_vdb .concept}

## 测试准备 {#section_e3n_syg_vdb .section}

1.  准备好测试程序的 Jar 包，假设名字为 mapreduce-examples.jar，本地存放路径为data\\resources。
2.  准备好 SleepJob 的测试资源。

    ```
    add jar data\resources\mapreduce-examples.jar -f;
    ```


## 测试步骤 {#section_rlv_bzg_vdb .section}

在 odpscmd 中执行 Sleep，如下所示：

```
  jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar 
  com.aliyun.odps.mapred.open.example.Sleep 10;
  jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar 
  com.aliyun.odps.mapred.open.example.Sleep 100;
```

## 预期结果 {#section_hzz_dzg_vdb .section}

作业成功结束后，对比不同 Sleep 时长的运行时间，可以看到效果。

## 代码示例 {#section_jgb_gzg_vdb .section}

```
package com.aliyun.odps.mapred.open.example;
import java.io.IOException;
import com.aliyun.odps.mapred.JobClient;
import com.aliyun.odps.mapred.MapperBase;
import com.aliyun.odps.mapred.conf.JobConf;
public class Sleep {
  private static final String SLEEP_SECS = "sleep.secs";
  public static class MapperClass extends MapperBase {
    @Override
    public void setup(TaskContext context) throws IOException {
      try {
        Thread.sleep(context.getJobConf().getInt(SLEEP_SECS, 1) * 1000);
      } catch (InterruptedException e) {
        throw new RuntimeException(e);
      }
    }
  }
  public static void main(String[] args) throws Exception {
    if (args.length != 1) {
      System.err.println("Usage: Sleep <sleep_secs>");
      System.exit(-1);
    }
    JobConf job = new JobConf();
    job.setMapperClass(MapperClass.class);
    job.setNumReduceTasks(0);
    job.setNumMapTasks(1);
    job.set(SLEEP_SECS, args[0]);
    JobClient.runJob(job);
  }
}

```

