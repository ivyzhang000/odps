# Java SDK {#concept_utw_vvc_5db .concept}

本文将为您介绍较为常用的 MaxCompute 核心接口，更多详情请参见 [SDK Java Doc](http://repo.aliyun.com/java-sdk-doc/)。

您可以通过 Maven 管理配置新 SDK 的版本。Maven 的配置信息如下（最新版本可以随时到 [search.maven.org](http://search.maven.org/?spm=5176.doc27991.2.1.BgYcC5) 搜索 odps-sdk-core 获取）：

```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>odps-sdk-core</artifactId>
  <version>0.26.2-public</version>
</dependency>
```

MaxCompute 提供的 SDK 包整体信息，如下表所示：

|包名|描述|
|:-|:-|
|odps-sdk-core|MaxCompute 的基础功能，例如：对表，Project 的操作，以及 Tunnel 均在此包中|
|odps-sdk-commons|一些 Util 封装|
|odps-sdk-udf|UDF 功能的主体接口|
|odps-sdk-mapred|MapReduce 功能|
|odps-sdk-graph|Graph Java SDK，搜索关键词“odps-sdk-graph”|

## AliyunAccount {#section_os1_f5b_wdb .section}

阿里云认证账号。输入参数为 accessId 及 accessKey，是阿里云用户的身份标识和认证密钥。此类用来初始化 MaxCompute。

## MaxCompute {#section_i25_f5b_wdb .section}

MaxCompute SDK 的入口，您可通过此类来获取项目空间下的所有对象集合，包括：[Projects](../cn.zh-CN/产品简介/基本概念/项目空间.md)，[Tables](../cn.zh-CN/产品简介/基本概念/表.md)，[Resources](../cn.zh-CN/产品简介/基本概念/资源.md)，[Functions](../cn.zh-CN/产品简介/基本概念/函数.md)，[Instances](../cn.zh-CN/产品简介/基本概念/任务实例.md)。

**说明：** MaxCompute 原名 ODPS，因此在现有的 SDK 版本中，入口类仍命名为 ODPS。

您可以通过传入 AliyunAccount 实例来构造 MaxCompute 对象。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Table t : odps.tables()) {
        ....
    }

```

## Projects {#section_eg4_k5b_wdb .section}

Projects 是 MaxCompute 中所有项目空间的集合。集合中的元素为 Project。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Project p = odps.projects().get("my_exists");
    p.reload();
    Map<String, String> properties = prj.getProperties();
    ...

```

## Project {#section_lg2_n5b_wdb .section}

Project 是对项目空间信息的描述，可以通过 Projects 获取相应的项目空间。

## SQLTask {#section_fpg_45b_wdb .section}

SQLTask 是用于运行、处理 SQL 任务的接口。可以通过 run 接口直接运行 SQL。\(**注意：每次只能提交运行一个SQL语句。**\)

run 接口返回 Instance 实例，通过 Instance 获取 SQL 的运行状态及运行结果。程序示例如下：

```
    import java.util.List;
    import com.aliyun.odps.Instance;
    import com.aliyun.odps.Odps;
    import com.aliyun.odps.OdpsException;
    import com.aliyun.odps.account.Account;
    import com.aliyun.odps.account.AliyunAccount;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.task.SQLTask;
    public class testSql {
    private static final String accessId = "";
    private static final String accessKey = "";
    private static final String endPoint = "http://service.odps.aliyun.com/api";
    private static final String project = "";
    private static final String sql = "select category from iris;";
    public static void
    main(String[] args) {
      Account account = new AliyunAccount(accessId, accessKey);
       Odps odps = new Odps(account);
       odps.setEndpoint(endPoint);
       odps.setDefaultProject(project);
       Instance i;
      try {
         i = SQLTask.run(odps, sql);
         i.waitForSuccess();
         List<Record> records = SQLTask.getResult(i); 
         for(Record r:records){
            System.out.println(r.get(0).toString());
         }
      } catch (OdpsException e) {
         e.printStackTrace();
      }
   }
  }

```

**说明：** 如果您想创建表，需要通过 SQLTask 接口，而不是 Table 接口。您需要将 [创建表](cn.zh-CN/用户指南/SQL/DDL语句.md)的语句传入 SQLTask。

## Instances {#section_xtz_s5b_wdb .section}

Instances 是 MaxCompute 中所有实例（Instance）的集合，集合中的元素为 Instance。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Instance i : odps.instances()) {
        ....
    }

```

## Instance {#section_rqp_v5b_wdb .section}

Instance 是对实例信息的描述，可以通过 Instances 获取相应的实例。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Instance ins = odps.instances().get("instance id");
    Date startTime = instance.getStartTime();
    Date endTime = instance.getEndTime();
    ...
    Status instanceStatus = instance.getStatus();
    String instanceStatusStr = null;
    if (instanceStatus == Status.TERMINATED) {
      instanceStatusStr = TaskStatus.Status.SUCCESS.toString();
      Map<String, TaskStatus> taskStatus = instance.getTaskStatus();
      for (Entry<String, TaskStatus> status : taskStatus.entrySet()) {
        if (status.getValue().getStatus() != TaskStatus.Status.SUCCESS) {
          instanceStatusStr = status.getValue().getStatus().toString();
          break;
        }
      }
    } else {
      instanceStatusStr = instanceStatus.toString();
    }
    ...
    TaskSummary summary = instance.getTaskSummary("instance name");
    String s = summary.getSummaryText();

```

## Tables {#section_bwd_z5b_wdb .section}

Tables 是 MaxCompute 中所有表的集合，集合中的元素为 Table。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Table t : odps.tables()) {
        ....
    }

```

## Table {#section_c13_bvb_wdb .section}

Table 是对表信息的描述，可以通过 Tables 获取相应的表。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Table t = odps.tables().get("table name");
    t.reload();
    Partition part = t.getPartition(new PartitionSpec(tableSpec[1]));
    part.reload();
    ...

```

## Resources {#section_tdn_dvb_wdb .section}

Resources 是 MaxCompute 中所有资源的集合。集合中的元素为 Resource。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Resource r : odps.resources()) {
        ....
    }

```

## Resource {#section_p3x_fvb_wdb .section}

Resource 是对资源信息的描述，可以通过 Resources 获取相应的资源。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Resource r = odps.resources().get("resource name");
    r.reload();
    if (r.getType() == Resource.Type.TABLE) {
     TableResource tr = new TableResource(r);
     String tableSource = tr.getSourceTable().getProject() + "."
         + tr.getSourceTable().getName();
     if (tr.getSourceTablePartition() != null) {
       tableSource += " partition(" + tr.getSourceTablePartition().toString()
           + ")";
     }
     ....   
    }

```

创建文件资源的示例，如下所示：

```
    String projectName = "my_porject";
    String source = "my_local_file.txt";
    File file = new File(source);
    InputStream is = new FileInputStream(file);
    FileResource resource = new FileResource();
    String name = file.getName();
    resource.setName(name);
    odps.resources().create(projectName, resource, is);
```

创建表资源的示例，如下所示：

```
    TableResource resource = new TableResource(tableName, tablePrj, partitionSpec);
    //resource.setName(INVALID_USER_TABLE);
    resource.setName("table_resource_name");
    odps.resources().update(projectName, resource);
```

## Functions {#section_b4d_lvb_wdb .section}

Functions 是 MaxCompute 中所有函数的集合。集合中的元素为 Function。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Function f : odps.functions()) {
        ....
    }

```

## Function {#section_xbw_nvb_wdb .section}

Function 是对函数信息的描述，可以通过 Functions 获取相应的函数。程序示例如下：

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Function f = odps.functions().get("function name");
    List<Resource> resources = f.getResources();

```

创建函数的示例，如下所示：

```
    String resources = "xxx:xxx";
    String classType = "com.aliyun.odps.mapred.open.example.WordCount";
    ArrayList<String> resourceList = new ArrayList<String>();
    for (String r : resources.split(":")) {
      resourceList.add(r);
    }
    Function func = new Function();
    func.setName(name);
    func.setClassType(classType);
    func.setResources(resourceList);
    odps.functions().create(projectName, func);

```

