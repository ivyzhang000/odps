# Logview {#concept_kkg_xgn_vdb .concept}

Logview 是 MaxCompute Job 提交后查看和 Debug 任务的工具。

通过 Logview 可看到一个 Job 的如下内容：

-   任务的运行状态。
-   任务的运行结果。
-   任务的细节和每个步骤的进度。

Job 提交到 MaxCompute 后，会生成 Logview 的链接，您可以直接在浏览器上打开 Logview 链接，进入查看 Job 的信息，每个 Job 的 Logview 页面的有效期为一周。

## Logview 功能组件 {#section_a35_tln_vdb .section}

下面结合具体的 Logview Web UI 界面，为您介绍每个组件的含义。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/2353_zh-CN.png)

Logview 的首页分成上下两部分：

-   Instance 信息。
-   Task 信息。

**Instance 信息**

Logview 首页中，上半部分是您提交的 SQL 生成的 MaxCompute Instance，每个 SQL 提交后会生成唯一的 ID。Latency 指运行总共消耗的时间，其他页面的 latency 含义类似。下面重点介绍 Status。

Status 包含四种状态：

-   Waiting：说明当前作业正在 MaxCompute 中处理，并没有提交到 Fuxi 中运行。

-   Waiting List : n 说明作业已提交到 Fuxi 并在 Fuxi 中排队，在队列中处于第 n 位。

-   Running：作业在 Fuxi 中运行。

-   Terminated：作业已结束，此时无队列信息。


其中非 Terminated 状态都可以单击查看详细队列信息。

单击 Status 查看队列详细信息：

-   Sub Status：表示当前子状态信息。
-   WaitPos：表示排队位置，如果是`0`表示正在运行，如果为`-`表示尚未到 Fuxi。
-   QueueLength：表示 Fuxi 中总的队列长度。
-   Total Priority：表示作业运行时经过系统判断后授予的优先级。
-   SubStatus History：单击后，可以查看作业执行的详细历史状态，包含状态码、状态描述、开始时间、持续时间等（某些版本暂时无历史信息）。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/2356_zh-CN.png)


**Task 信息**

Logview 首页中，下半部分是该 task 的说明，下面重点说明 Result 与 Detail。

Result：

在 Job 运行结束后，可以看到运行结果，如一条 select SQL 的结果，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/2365_zh-CN.png)

Detail：

一个 Job 在运行中和结束后，均可以单击 **Detail**来查看任务运行的具体情况。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/2366_zh-CN.png)

-   一个 MaxCompute Task 由一个或者多个 Fuxi Job 组成。例如当您的 SQL 任务十分复杂时，MaxCompute 会向 Fuxi 提交多个 Fuxi Job。
-   每个 Fuxi Job 由一个或者多个 Fuxi Task 组成。简单的 MapReduce 通常会产生两个 Fuxi Task，一个是 Map，一个是 Reduce，您会见到两个 Fuxi Task 的名字分别为 M1 和 R2，当 SQL 比较复杂时，可能会产生多个 Fuxi Task，如上图所示。
-   在每个 Task 中，可以看到 Task 的名字，对于 M1，表示这是一个 Map task，R5\_4 中的 4 表示它依赖 J4 执行结束才能开始执行。同理，J4\_1\_2\_3 表示 Join4 这个阶段要依赖 M1、M2、M3 三个 task 完全成才能启动运行。
-   I/O Records 表示这个 task 的输入和输出的 records 数。

单击任一 Fuxi Task，可查看 Fuxi Instance 内容，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/2367_zh-CN.png)

每个 Fuxi Task 由一个或者多个 Fuxi Instance 组成，当您的输入数据量变大时，MaxCompute 会在每个 Task 启动更多的节点来处理数据。每个节点就是一个 Fuxi Instance。双击 Fuxi Task 最右边一栏 **查看**，或者直接双击该行，就可以打开具体的 Fuxi Instance 信息。

在页面的下方，Logview 为不同阶段的 Instance 进行了分组，查看出错的节点可以选择 Failed 栏。

在 StdOut 和 StdErr 两栏中，可以查看标准输出和标准错误信息，您自己打印的信息也可以在这里查看。

## 通过 Logview 进行问题排查 {#section_xf2_1pn_vdb .section}

-   **出错的任务**

    当有任务出错时，您可以在 Logview 页面的 Result 中看到错误的提示信息，也可以在 Detail 页面中通过 Fuxi Instance 的 stderr，查看具体某个 Instance 出错的信息。

-   **数据倾斜**

    运行缓慢有时是由于在某个 Fuxi Task 的所有 Fuxi Instance 中，有个别 Instance 形成长尾造成，长尾的现象就是同一个 Task 内任务分配不均。这时可以在任务运行完后，在 Summay 标签页中看运行结果。在每个 Task 中都可以看到形如这样的输出：

    ```
    output records:
    R2_1_Stg1: 199998999  (min: 22552459, max: 177446540, avg: 99999499)
    ```

    在这里如果看到 min 和 max 相差很大，就说明在这一阶段出现了数据倾斜。比如在 Join 时，某个字段中有一个值出现的比例很高，在这一字段上做 Join 就会出现数据倾斜。


