# K-均值聚类 {#concept_svw_slm_vdb .concept}

k-均值聚类（Kmeans） 算法是非常基础并大量使用的聚类算法。

算法基本原理：以空间中 k 个点为中心进行聚类，对最靠近它们的点进行归类。通过迭代的方法，逐次更新各聚类中心的值，直至得到最好的聚类结果。

假设要把样本集分为 k 个类别，算法描述如下：

1.  适当选择 k 个类的初始中心。
2.  在第 i 次迭代中，对任意一个样本，求其到 k 个中心的距离，将该样本归到距离最短的中心所在的类。
3.  利用均值等方法更新该类的中心值。
4.  对于所有的 k 个聚类中心，如果利用上两步的迭代法更新后，值保持不变或者小于某个阈值，则迭代结束，否则继续迭代。

## 代码示例 {#section_zmw_pxm_vdb .section}

K-均值聚类算法的代码，如下所示：

```
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.log4j.Logger;

import com.aliyun.odps.io.WritableRecord;
import com.aliyun.odps.graph.Aggregator;
import com.aliyun.odps.graph.ComputeContext;
import com.aliyun.odps.graph.GraphJob;
import com.aliyun.odps.graph.GraphLoader;
import com.aliyun.odps.graph.MutationContext;
import com.aliyun.odps.graph.Vertex;
import com.aliyun.odps.graph.WorkerContext;
import com.aliyun.odps.io.DoubleWritable;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.io.NullWritable;
import com.aliyun.odps.data.TableInfo;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.io.Tuple;
import com.aliyun.odps.io.Writable;

public class Kmeans {
  private final static Logger LOG = Logger.getLogger(Kmeans.class);

  public static class KmeansVertex extends
      Vertex<Text, Tuple, NullWritable, NullWritable> {

    @Override
    public void compute(
        ComputeContext<Text, Tuple, NullWritable, NullWritable> context,
        Iterable<NullWritable> messages) throws IOException {
      context.aggregate(getValue());
    }

  }

  public static class KmeansVertexReader extends
      GraphLoader<Text, Tuple, NullWritable, NullWritable> {
    @Override
    public void load(LongWritable recordNum, WritableRecord record,
        MutationContext<Text, Tuple, NullWritable, NullWritable> context)
        throws IOException {
      KmeansVertex vertex = new KmeansVertex();
      vertex.setId(new Text(String.valueOf(recordNum.get())));
      vertex.setValue(new Tuple(record.getAll()));
      context.addVertexRequest(vertex);
    }

  }

  public static class KmeansAggrValue implements Writable {

    Tuple centers = new Tuple();
    Tuple sums = new Tuple();
    Tuple counts = new Tuple();

    @Override
    public void write(DataOutput out) throws IOException {
      centers.write(out);
      sums.write(out);
      counts.write(out);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
      centers = new Tuple();
      centers.readFields(in);
      sums = new Tuple();
      sums.readFields(in);
      counts = new Tuple();
      counts.readFields(in);
    }

    @Override
    public String toString() {
      return "centers " + centers.toString() + ", sums " + sums.toString()
          + ", counts " + counts.toString();
    }

  }

  public static class KmeansAggregator extends Aggregator<KmeansAggrValue> {

    @SuppressWarnings("rawtypes")
    @Override
    public KmeansAggrValue createInitialValue(WorkerContext context)
        throws IOException {
      KmeansAggrValue aggrVal = null;
      if (context.getSuperstep() == 0) {
        aggrVal = new KmeansAggrValue();
        aggrVal.centers = new Tuple();
        aggrVal.sums = new Tuple();
        aggrVal.counts = new Tuple();

        byte[] centers = context.readCacheFile("centers");
        String lines[] = new String(centers).split("\n");

        for (int i = 0; i < lines.length; i++) {
          String[] ss = lines[i].split(",");
          Tuple center = new Tuple();
          Tuple sum = new Tuple();
          for (int j = 0; j < ss.length; ++j) {
            center.append(new DoubleWritable(Double.valueOf(ss[j].trim())));
            sum.append(new DoubleWritable(0.0));
          }
          LongWritable count = new LongWritable(0);
          aggrVal.sums.append(sum);
          aggrVal.counts.append(count);
          aggrVal.centers.append(center);
        }
      } else {
        aggrVal = (KmeansAggrValue) context.getLastAggregatedValue(0);
      }

      return aggrVal;
    }

    @Override
    public void aggregate(KmeansAggrValue value, Object item) {
      int min = 0;
      double mindist = Double.MAX_VALUE;
      Tuple point = (Tuple) item;

      for (int i = 0; i < value.centers.size(); i++) {
        Tuple center = (Tuple) value.centers.get(i);
        // use Euclidean Distance, no need to calculate sqrt
        double dist = 0.0d;
        for (int j = 0; j < center.size(); j++) {
          double v = ((DoubleWritable) point.get(j)).get()
              - ((DoubleWritable) center.get(j)).get();
          dist += v * v;
        }
        if (dist < mindist) {
          mindist = dist;
          min = i;
        }
      }

      // update sum and count
      Tuple sum = (Tuple) value.sums.get(min);
      for (int i = 0; i < point.size(); i++) {
        DoubleWritable s = (DoubleWritable) sum.get(i);
        s.set(s.get() + ((DoubleWritable) point.get(i)).get());
      }
      LongWritable count = (LongWritable) value.counts.get(min);
      count.set(count.get() + 1);
    }

    @Override
    public void merge(KmeansAggrValue value, KmeansAggrValue partial) {
      for (int i = 0; i < value.sums.size(); i++) {
        Tuple sum = (Tuple) value.sums.get(i);
        Tuple that = (Tuple) partial.sums.get(i);
        for (int j = 0; j < sum.size(); j++) {
          DoubleWritable s = (DoubleWritable) sum.get(j);
          s.set(s.get() + ((DoubleWritable) that.get(j)).get());
        }
      }

      for (int i = 0; i < value.counts.size(); i++) {
        LongWritable count = (LongWritable) value.counts.get(i);
        count.set(count.get() + ((LongWritable) partial.counts.get(i)).get());
      }
    }

    @SuppressWarnings("rawtypes")
    @Override
    public boolean terminate(WorkerContext context, KmeansAggrValue value)
        throws IOException {

      // compute new centers
      Tuple newCenters = new Tuple(value.sums.size());
      for (int i = 0; i < value.sums.size(); i++) {
        Tuple sum = (Tuple) value.sums.get(i);
        Tuple newCenter = new Tuple(sum.size());
        LongWritable c = (LongWritable) value.counts.get(i);
        for (int j = 0; j < sum.size(); j++) {

          DoubleWritable s = (DoubleWritable) sum.get(j);
          double val = s.get() / c.get();
          newCenter.set(j, new DoubleWritable(val));

          // reset sum for next iteration
          s.set(0.0d);
        }
        // reset count for next iteration
        c.set(0);
        newCenters.set(i, newCenter);
      }

      // update centers
      Tuple oldCenters = value.centers;
      value.centers = newCenters;

      LOG.info("old centers: " + oldCenters + ", new centers: " + newCenters);

      // compare new/old centers
      boolean converged = true;
      for (int i = 0; i < value.centers.size() && converged; i++) {
        Tuple oldCenter = (Tuple) oldCenters.get(i);
        Tuple newCenter = (Tuple) newCenters.get(i);
        double sum = 0.0d;
        for (int j = 0; j < newCenter.size(); j++) {
          double v = ((DoubleWritable) newCenter.get(j)).get()
              - ((DoubleWritable) oldCenter.get(j)).get();
          sum += v * v;
        }
        double dist = Math.sqrt(sum);
        LOG.info("old center: " + oldCenter + ", new center: " + newCenter
            + ", dist: " + dist);
        // converge threshold for each center: 0.05
        converged = dist < 0.05d;
      }

      if (converged || context.getSuperstep() == context.getMaxIteration() - 1) {
        // converged or reach max iteration, output centers
        for (int i = 0; i < value.centers.size(); i++) {
          context.write(((Tuple) value.centers.get(i)).toArray());
        }
        // true means to terminate iteration
        return true;
      }

      // false means to continue iteration
      return false;
    }
  }

  private static void printUsage() {
    System.out.println("Usage: <in> <out> [Max iterations (default 30)]");
    System.exit(-1);
  }

  public static void main(String[] args) throws IOException {
    if (args.length < 2)
      printUsage();

    GraphJob job = new GraphJob();

    job.setGraphLoaderClass(KmeansVertexReader.class);
    job.setRuntimePartitioning(false);
    job.setVertexClass(KmeansVertex.class);
    job.setAggregatorClass(KmeansAggregator.class);
    job.addInput(TableInfo.builder().tableName(args[0]).build());
    job.addOutput(TableInfo.builder().tableName(args[1]).build());

    // default max iteration is 30
    job.setMaxIteration(30);
    if (args.length >= 3)
      job.setMaxIteration(Integer.parseInt(args[2]));

    long start = System.currentTimeMillis();
    job.run();
    System.out.println("Job Finished in "
        + (System.currentTimeMillis() - start) / 1000.0 + " seconds");
  }
}

```

上述代码，说明如下：

-   第 26 行：定义 KmeansVertex，compute\(\) 方法非常简单，只是调用上下文对象的 aggregate 方法，传入当前点的取值（Tuple 类型，向量表示）。
-   第 38 行：定义 KmeansVertexReader 类，加载图，将表中每一条记录解析为一个点，点标识无关紧要，这里取传入的 recordNum 序号作为标识，点值为记录的所有列组成的 Tuple。
-   第 83 行：定义 KmeansAggregator，这个类封装了 Kmeans 算法的主要逻辑，其中：
    -   createInitialValue 为每一轮迭代创建初始值（k 类中心点），若是第一轮迭代（superstep=0），该取值为初始中心点，否则取值为上一轮结束时的新中心点。
    -   aggregate 方法为每个点计算其到各个类中心的距离，并归为距离最短的类，并更新该类的 sum 和 count。
    -   merge 方法合并来自各个 worker 收集的 sum 和 count。
    -   terminate 方法根据各个类的 sum 和 count 计算新的中心点，若新中心点与之前的中心点距离小于某个阈值或者迭代次数到达最大迭代次数设置，则终止迭代（返回 false），写最终的中心点到结果表。
-   第 236 行：主程序（main 函数），定义 GraphJob，指定 Vertex/GraphLoader/Aggregator 等的实现，以及最大迭代次数（默认 30），并指定输入输出表。
-   第 243 行：job.setRuntimePartitioning\(false\)，对于 Kmeans 算法，加载图是不需要进行点的分发，设置 RuntimePartitioning 为 false，以提升加载图时的性能。

