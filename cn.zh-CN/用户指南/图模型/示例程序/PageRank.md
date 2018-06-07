# PageRank {#concept_jvb_slm_vdb .concept}

PageRank 算法是计算网页排名的经典算法：输入是一个有向图 G，其中顶点表示网页，如果存在网页 A 到网页 B 的链接，那么存在连接 A 到 B 的边。

算法基本原理，如下所示：

-   初始化：点值表示 PageRank 的 rank 值（double 类型），初始时，所有点取值为 1/TotalNumVertices。
-   迭代公式：PageRank\(i\)=0.15/TotalNumVertices+0.85\*sum，其中 sum 为所有指向 i 点的点（设为 j） PageRank\(j\)/out\_degree\(j\) 的累加值。

由算法基本原理可以看出，此算法非常适合使用 MaxCompute Graph 程序进行求解：每个点 j 维护其 PageRank 值，每一轮迭代都将 PageRank\(j\)/out\_degree\(j\) 发给其邻接点（向其投票），下一轮迭代时，每个点根据迭代公式重新计算 PageRank 取值。

## 代码示例 {#section_jlb_4wm_vdb .section}

```
import java.io.IOException;

import org.apache.log4j.Logger;

import com.aliyun.odps.io.WritableRecord;
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
import com.aliyun.odps.io.Writable;

public class PageRank {

  private final static Logger LOG = Logger.getLogger(PageRank.class);

  public static class PageRankVertex extends
      Vertex<Text, DoubleWritable, NullWritable, DoubleWritable> {

    @Override
    public void compute(
        ComputeContext<Text, DoubleWritable, NullWritable, DoubleWritable> context,
        Iterable<DoubleWritable> messages) throws IOException {
      if (context.getSuperstep() == 0) {
        setValue(new DoubleWritable(1.0 / context.getTotalNumVertices()));
      } else if (context.getSuperstep() >= 1) {
        double sum = 0;
        for (DoubleWritable msg : messages) {
          sum += msg.get();
        }
        DoubleWritable vertexValue = new DoubleWritable(
            (0.15f / context.getTotalNumVertices()) + 0.85f * sum);
        setValue(vertexValue);
      }
      if (hasEdges()) {
        context.sendMessageToNeighbors(this, new DoubleWritable(getValue()
            .get() / getEdges().size()));
      }
    }

    @Override
    public void cleanup(
        WorkerContext<Text, DoubleWritable, NullWritable, DoubleWritable> context)
        throws IOException {
      context.write(getId(), getValue());
    }
  }

  public static class PageRankVertexReader extends
      GraphLoader<Text, DoubleWritable, NullWritable, DoubleWritable> {

    @Override
    public void load(
        LongWritable recordNum,
        WritableRecord record,
        MutationContext<Text, DoubleWritable, NullWritable, DoubleWritable> context)
        throws IOException {
      PageRankVertex vertex = new PageRankVertex();
      vertex.setValue(new DoubleWritable(0));
      vertex.setId((Text) record.get(0));
      System.out.println(record.get(0));

      for (int i = 1; i < record.size(); i++) {
        Writable edge = record.get(i);
        System.out.println(edge.toString());
        if (!(edge.equals(NullWritable.get()))) {
          vertex.addEdge(new Text(edge.toString()), NullWritable.get());
        }
      }
      LOG.info("vertex edgs size: "
          + (vertex.hasEdges() ? vertex.getEdges().size() : 0));
      context.addVertexRequest(vertex);
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

    job.setGraphLoaderClass(PageRankVertexReader.class);
    job.setVertexClass(PageRankVertex.class);
    job.addInput(TableInfo.builder().tableName(args[0]).build());
    job.addOutput(TableInfo.builder().tableName(args[1]).build());

    // default max iteration is 30
    job.setMaxIteration(30);
    if (args.length >= 3)
      job.setMaxIteration(Integer.parseInt(args[2]));

    long startTime = System.currentTimeMillis();
    job.run();
    System.out.println("Job Finished in "
        + (System.currentTimeMillis() - startTime) / 1000.0 + " seconds");
  }
}

```

上述代码，说明如下：

-   第 23 行：定义 PageRankVertex ，其中：
    -   点值表示该点（网页）的当前 PageRank 取值。
    -   compute\(\) 方法使用迭代公式：`PageRank(i)=0.15/TotalNumVertices+0.85*sum`更新点值。
    -   cleanup\(\) 方法把点及其 PageRank 取值写到结果表中。
-   第 55 行：定义 PageRankVertexReader 类，加载图，将表中每一条记录解析为一个点，记录的第一列是起点，其他列为终点。
-   第 88 行：主程序（main 函数），定义 GraphJob，指定 Vertex/GraphLoader 等的实现，以及最大迭代次数（默认 30），并指定输入输出表。

