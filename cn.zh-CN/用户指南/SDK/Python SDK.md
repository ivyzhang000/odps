# Python SDK {#concept_g14_4wb_wdb .concept}

PyODPS是MaxCompute的Python版本的SDK，它提供了对MaxCompute对象的基本操作和DataFrame框架，可以轻松地在MaxCompute上进行数据分析。更多详情请参见[Github项目](https://github.com/aliyun/aliyun-odps-python-sdk)和包括所有接口、类的细节等内容的[详细文档](http://pyodps.readthedocs.org/)。

-   关于PyODPS的更多详情请参见[PyODPS云栖社区专辑](https://yq.aliyun.com/album/19)。

-   欢迎各位开发者参与到PyODPS的生态开发中，详情请参见[GitHub文档](http://pyodps.readthedocs.io/zh_CN/latest/?spm=a2c4e.11153959.blogcont138752.16.5bec51d32BpKgB)。

-   欢迎提交issue和merge request，加快PyODPS生态成长，更多详情请参见[代码](https://github.com/aliyun/aliyun-odps-python-sdk?spm=a2c4e.11153959.blogcont138752.17.5bec51d3IMNtLJ)。

-   钉钉技术交流群：**11701793**


## 安装 {#section_mj1_jxb_wdb .section}

PyODPS支持Python2.6以上（包括Python3），系统安装pip后，只需运行`pip install pyodps`，PyODPS的相关依赖便会自动安装。

## 快速开始 {#section_bqy_jxb_wdb .section}

首先，用阿里云账号初始化一个MaxCompute的入口，如下所示：

```
from odps import ODPS
odps = ODPS('**your-access-id**', '**your-secret-access-key**', '**your-default-project**',
            endpoint='**your-end-point**')
```

根据上述操作初始化后，便可对表、资源、函数等进行操作。

## 项目空间 {#section_sc2_nxb_wdb .section}

项目空间是MaxCompute的基本组织单元，类似于Database的概念。

您可通过`get_project`获取到某个项目空间，如下所示：

```
project = odps.get_project('my_project')  # 取到某个项目
project = odps.get_project()              # 取到默认项目
```

**说明：** 

-   如果不提供参数，则获取到默认项目空间。
-   通过`exist_project`，可以查看某个项目空间是否存在。

-   表是MaxCompute的数据存储单元。


## 表操作 {#section_mmv_kyb_wdb .section}

通过调用`list_tables`可以列出项目空间下的所有表，如下所示：

```
for table in odps.list_tables():
    # 处理每张表
```

通过调用`exist_table`可以判断表是否存在，通过调用`get_table`可以获取表。

```
>>> t = odps.get_table('dual')
>>> t.schema
odps.Schema {
  c_int_a                 bigint
  c_int_b                 bigint
  c_double_a              double
  c_double_b              double
  c_string_a              string
  c_string_b              string
  c_bool_a                boolean
  c_bool_b                boolean
  c_datetime_a            datetime
  c_datetime_b            datetime
}
>>> t.lifecycle
-1
>>> print(t.creation_time)
2014-05-15 14:58:43
>>> t.is_virtual_view
False
>>> t.size
1408
>>> t.schema.columns
[<column c_int_a, type bigint>,
 <column c_int_b, type bigint>,
 <column c_double_a, type double>,
 <column c_double_b, type double>,
 <column c_string_a, type string>,
 <column c_string_b, type string>,
 <column c_bool_a, type boolean>,
 <column c_bool_b, type boolean>,
 <column c_datetime_a, type datetime>,
 <column c_datetime_b, type datetime>]

```

## 创建表的Schema {#section_hxy_qyb_wdb .section}

初始化的方法有两种，如下所示：

-   通过表的列和可选的分区来初始化。

    ```
    >>> from odps.models import Schema, Column, Partition
    >>> columns = [Column(name='num', type='bigint', comment='the column')]
    >>> partitions = [Partition(name='pt', type='string', comment='the partition')]
    >>> schema = Schema(columns=columns, partitions=partitions)
    >>> schema.columns
    [<column num, type bigint>, <partition pt, type string>]
    
    ```

-   通过调用`Schema.from_lists`，虽然调用更加方便，但显然无法直接设置列和分区的注释。

    ```
    >>> schema = Schema.from_lists(['num'], ['bigint'], ['pt'], ['string'])
    >>> schema.columns
    [<column num, type bigint>, <partition pt, type string>]
    
    ```


## 创建表 {#section_r2m_wyb_wdb .section}

您可以使用表的Schema来创建表，操作如下所示：

```
>>> table = odps.create_table('my_new_table', schema)
>>> table = odps.create_table('my_new_table', schema, if_not_exists=True)  # 只有不存在表时才创建
>>> table = o.create_table('my_new_table', schema, lifecycle=7)  # 设置生命周期
```

也可以使用逗号连接的**字段名 字段类型**字符串组合来创建表，操作如下所示：

```
>>> # 创建非分区表
>>> table = o.create_table('my_new_table', 'num bigint, num2 double', if_not_exists=True)
>>> # 创建分区表可传入 (表字段列表, 分区字段列表)
>>> table = o.create_table('my_new_table', ('num bigint, num2 double', 'pt string'), if_not_exists=True)
```

在未经设置的情况下，创建表时，只允许使用bigint、double、decimal、string、datetime、boolean、map和array类型。

如果您的服务位于公共云，或者支持tinyint、struct等新类型，可以设置`options.sql.use_odps2_extension = True`，以打开这些类型的支持，示例如下：

```
>>> from odps import options
>>> options.sql.use_odps2_extension = True
>>> table = o.create_table('my_new_table', 'cat smallint, content struct<title:varchar(100), body string>')

```

## 获取表数据 {#section_cwv_czb_wdb .section}

您可通过以下三种方法获取表数据。

-   通过调用`head`获取表数据，但仅限于查看每张表开始的小于1万条的数据，如下所示：

    ```
    >>> t = odps.get_table('dual')
    >>> for record in t.head(3):
    >>>     print(record[0])  # 取第0个位置的值
    >>>     print(record['c_double_a'])  # 通过字段取值
    >>>     print(record[0: 3])  # 切片操作
    >>>     print(record[0, 2, 3])  # 取多个位置的值
    >>>     print(record['c_int_a', 'c_double_a'])  # 通过多个字段取值
    ```

-   通过在table上执行`open_reader`操作，打开一个reader来读取数据。您可以使用with表达式，也可以不使用。

    ```
    # 使用with表达式
    >>> with t.open_reader(partition='pt=test') as reader:
    >>>     count = reader.count
    >>>     for record in reader[5:10]  # 可以执行多次，直到将count数量的record读完，这里可以改造成并行操作
    >>>         # 处理一条记录
    >>>
    >>> # 不使用with表达式
    >>> reader = t.open_reader(partition='pt=test')
    >>> count = reader.count
    >>> for record in reader[5:10]
    >>>     # 处理一条记录
    ```

-   通过使用Tunnel API读取表数据，`open_reader`操作其实也是对Tunnel API的封装。

## 写入数据 {#section_c5z_jzb_wdb .section}

类似于`open_reader`，table对象同样可以执行`open_writer`来打开writer，并写数据。如下所示：

```
>>> # 使用 with 表达式
>>> with t.open_writer(partition='pt=test') as writer:
>>>     writer.write(records)  # 这里records可以是任意可迭代的records，默认写到block 0
>>>
>>> with t.open_writer(partition='pt=test', blocks=[0, 1]) as writer:  # 这里同是打开两个block
>>>     writer.write(0, gen_records(block=0))
>>>     writer.write(1, gen_records(block=1))  # 这里两个写操作可以多线程并行，各个block间是独立的
>>>
>>> # 不使用 with 表达式
>>> writer = t.open_writer(partition='pt=test', blocks=[0, 1])
>>> writer.write(0, gen_records(block=0))
>>> writer.write(1, gen_records(block=1))
>>> writer.close()  # 不要忘记关闭 writer，否则数据可能写入不完全
```

同样，向表中写入数据也是对Tunnel API的封装，更多详情请参见[数据上传下载通道](cn.zh-CN/用户指南/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md)。

## 删除表 {#section_ayt_mzb_wdb .section}

删除表的操作，如下所示：

```
>>> odps.delete_table('my_table_name', if_exists=True)  #  只有表存在时删除
>>> t.drop()  # Table对象存在的时候可以直接执行drop函数
```

## 表分区 {#section_vxt_4zb_wdb .section}

-   **基本操作**

    遍历表的全部分区，如下所示：

    ```
    >>> for partition in table.partitions:
    >>>     print(partition.name)
    >>> for partition in table.iterate_partitions(spec='pt=test'):
    >>>     # 遍历二级分区
    ```

    判断分区是否存在，如下所示：

    ```
    >>> table.exist_partition('pt=test,sub=2015')
    ```

    获取分区，如下所示：

    ```
    >>> partition = table.get_partition('pt=test')
    >>> print(partition.creation_time)
    2015-11-18 22:22:27
    >>> partition.size
    0
    ```

-   **创建分区**

    ```
    >>> t.create_partition('pt=test', if_not_exists=True) # 不存在的时候才创建
    ```

-   **删除分区**

    ```
    >>> t.delete_partition('pt=test', if_exists=True)  # 存在的时候才删除
    >>> partition.drop()  # Partition对象存在的时候直接drop
    ```


## SQL {#section_hhn_xzb_wdb .section}

PyODPS支持MaxCompute SQL的查询，并可以读取执行的结果。

-   **执行SQL**

    ```
    >>> odps.execute_sql('select * from dual')  #  同步的方式执行，会阻塞直到SQL执行完成
    >>> instance = odps.run_sql('select * from dual')  # 异步的方式执行
    >>> instance.wait_for_success()  # 阻塞直到完成
    ```

-   **读取SQL执行结果**

    运行SQL的instance能够直接执行`open_reader`的操作，一种情况是SQL返回了结构化的数据。

    ```
    >>> with odps.execute_sql('select * from dual').open_reader() as reader:
    >>>     for record in reader:
    >>>         # 处理每一个record
    ```


另一种情况是SQL可能执行的比如`desc`，这时通过`reader.raw`属性取到原始的SQL执行结果。

```
>>> with odps.execute_sql('desc dual').open_reader() as reader:
>>>     print(reader.raw)
```

## Resource {#section_xvk_g1c_wdb .section}

资源在MaxCompute上常用在UDF和MapReduce中。

列出所有资源还是可以使用`list_resources`，判断资源是否存在使用`exist_resource`。删除资源时，可以调用`delete_resource`，或者直接对于Resource对象调用`drop`方法。

在PyODPS中，主要支持两种资源类型，一种是文件，另一种是表。

-   **文件资源**

    文件资源包括基础的`file`类型、以及`py`、`jar`和`archive`。

    **说明：** 在DataWorks中，py格式的文件资源请以file形式上传，详情请参见[Python UDF文档](https://yq.aliyun.com/articles/300307)。

    **创建文件资源**

    创建文件资源可以通过给定资源名、文件类型、以及一个file-like的对象（或者是字符串对象）来创建，示例如下：

    ```
    resource = odps.create_resource('test_file_resource', 'file', file_obj=open('/to/path/file'))  # 使用file-like的对象
    resource = odps.create_resource('test_py_resource', 'py', file_obj='import this')  # 使用字符串
    ```

    **读取和修改文件资源**

    对文件资源调用`open`方法，或者在MaxCompute入口调用`open_resource`都能打开一个资源， 打开后的对象会是file-like的对象。类似于Python内置的`open`方法，文件资源也支持打开的模式。示例如下：

    ```
    >>> with resource.open('r') as fp:  # 以读模式打开
    >>>     content = fp.read()  # 读取全部的内容
    >>>     fp.seek(0)  # 回到资源开头
    >>>     lines = fp.readlines()  # 读成多行
    >>>     fp.write('Hello World')  # 报错，读模式下无法写资源
    >>>
    >>> with odps.open_resource('test_file_resource', mode='r+') as fp:  # 读写模式打开
    >>>     fp.read()
    >>>     fp.tell()  # 当前位置
    >>>     fp.seek(10)
    >>>     fp.truncate()  # 截断后面的内容
    >>>     fp.writelines(['Hello\n', 'World\n'])  # 写入多行
    >>>     fp.write('Hello World')
    >>>     fp.flush()  # 手动调用会将更新提交到ODPS
    ```

    所有支持的打开类型包括：

    -   `r`：读模式，只能打开不能写。
    -   `w`：写模式，只能写入而不能读文件，注意用写模式打开，文件内容会被先清空。
    -   `a`：追加模式，只能写入内容到文件末尾。
    -   `r+`：读写模式，能任意读写内容。
    -   `w+`：类似于`r+`，但会先清空文件内容。
    -   `a+`：类似于`r+`，但写入时只能写入文件末尾。
    同时，PyODPS中，文件资源支持以二进制模式打开，打开如说一些压缩文件等等就需要以这种模式， 因此`rb`就是指以二进制读模式打开文件，`r+b`是指以二进制读写模式打开。

-   **表资源**

    **创建表资源**

    ```
    >>> odps.create_resource('test_table_resource', 'table', table_name='my_table', partition='pt=test')
    ```

    **更新表资源**

    ```
    >>> table_resource = odps.get_resource('test_table_resource')
    >>> table_resource.update(partition='pt=test2', project_name='my_project2')
    ```


## DataFrame {#section_ns3_1bc_wdb .section}

PyODPS提供了DataFrame API，它提供了类似pandas的接口，但是能充分利用MaxCompute的计算能力。完整的DataFrame文档请参见[DataFrame](http://pyodps.readthedocs.io/zh_CN/latest/df.html)。

DataFrame的示例如下：

**说明：** 在所有步骤开始前，需要创建MaxCompute对象。

```
>>> o = ODPS('**your-access-id**', '**your-secret-access-key**',
             project='**your-project**', endpoint='**your-end-point**'))
```

此处以movielens 100K作为示例，假设已经有三张表，分别是`pyodps_ml_100k_movies`（电影相关的数据），`pyodps_ml_100k_users`（用户相关的数据），`pyodps_ml_100k_ratings`（评分有关的数据）。

只需传入Table对象，便可创建一个DataFrame对象。如下所示：

```
>>> from odps.df import DataFrame
```

```
>>> users = DataFrame(o.get_table('pyodps_ml_100k_users'))
```

通过dtypes属性来查看这个DataFrame有哪些字段，分别是什么类型，如下所示：

```
>>> users.dtypes
```

通过head方法，可以获取前N条数据，方便快速预览数据。如下所示：

```
>>> users.head(10)
```

|　|user\_id|age|sex|occupation|zip\_code|
|:-|:-------|:--|:--|:---------|:--------|
|0|1|24|M|technician|85711|
|1|2|53|F|other|94043|
|2|3|23|M|writer|32067|
|3|4|24|M|technician|43537|
|4|5|33|F|other|15213|
|5|6|42|M|executive|98101|
|6|7|57|M|administrator|91344|
|7|8|36|M|administrator|05201|
|8|9|29|M|student|01002|
|9|10|53|M|lawyer|90703|

有时候，并不需要都看到所有字段，便可以从中筛选出一部分。如下所示：

```
>>> users[['user_id', 'age']].head(5)
```

|　|user\_id|age|
|:-|:-------|:--|
|0|1|24|
|1|2|53|
|2|3|23|
|3|4|24|
|4|5|33|

有时候只是排除个别字段。如下所示：

```
>>> users.exclude('zip_code', 'age').head(5)
```

|　|user\_id|sex|occupation|
|:-|:-------|:--|:---------|
|0|1|M|technician|
|1|2|F|other|
|2|3|M|writer|
|3|4|M|technician|
|4|5|F|other|

排除掉一些字段的同时，想要通过计算得到一些新的列，比如将sex为M的置为True，否则为False，并取名叫sex\_bool。如下所示：

```
>>> users.select(users.exclude('zip_code', 'sex'), sex_bool=users.sex == 'M').head(5)
```

|　|user\_id|age|occupation|sex\_bool|
|:-|:-------|:--|:---------|:--------|
|0|1|24|technician|True|
|1|2|53|other|False|
|2|3|23|writer|True|
|3|4|24|technician|True|
|4|5|33|other|False|

若想知道年龄在20到25岁之间的人有多少个，如下所示：

```
>>> users.age.between(20, 25).count().rename('count')
943
```

若想知道男女用户分别有多少，如下所示：

```
>>> users.groupby(users.sex).count()
```

|　|sex|count|
|:-|:--|:----|
|0|F|273|
|1|M|670|

若想将用户按职业划分，从高到底，获取人数最多的前10个职业，如下所示：

```
>>> df = users.groupby('occupation').agg(count=users['occupation'].count())
>>> df.sort(df['count'], ascending=False)[:10]
```

|　|occupation|count|
|:-|:---------|:----|
|0|student|196|
|1|other|105|
|2|educator|95|
|3|administrator|79|
|4|engineer|67|
|5|programmer|66|
|6|librarian|51|
|7|writer|45|
|8|executive|32|
|9|scientist|31|

DataFrame API提供了value\_counts方法来快速达到同样的目的。如下所示：

```
>>> users.occupation.value_counts()[:10]
```

|　|occupation|count|
|:-|:---------|:----|
|0|student|196|
|1|other|105|
|2|educator|95|
|3|administrator|79|
|4|engineer|67|
|5|programmer|66|
|6|librarian|51|
|7|writer|45|
|8|executive|32|
|9|scientist|31|

使用更直观的图来查看这份数据，如下所示：

```
>>> %matplotlib inline
```

使用横向的柱状图来可视化，如下所示：

```
>>> users['occupation'].value_counts().plot(kind='barh', x='occupation', 
ylabel='prefession')
```

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12065/2854_zh-CN.png)

将年龄分成30组，查看各年龄分布的直方图，如下所示：

```
>>> users.age.hist(bins=30, title="Distribution of users' ages", xlabel='age', ylabel='count of users')
```

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12065/2855_zh-CN.png)

使用join把这三张表进行联合后，把它保存成一张新的表。如下所示：

```
>>> movies = DataFrame(o.get_table('pyodps_ml_100k_movies'))
>>> ratings = DataFrame(o.get_table('pyodps_ml_100k_ratings'))
>>> o.delete_table('pyodps_ml_100k_lens', if_exists=True)
>>> lens = movies.join(ratings).join(users).persist('pyodps_ml_100k_lens')
>>> lens.dtypes
```

```
odps.Schema {
  movie_id                            int64
  title                               string
  release_date                        string
  video_release_date                  string
  imdb_url                            string
  user_id                             int64
  rating                              int64
  unix_timestamp                      int64
  age                                 int64
  sex                                 string
  occupation                          string
  zip_code                            string
}
```

把0到80岁的年龄，分成8个年龄段，如下所示：

```
>>> labels = ['0-9', '10-19', '20-29', '30-39', '40-49', '50-59', '60-69', '70-79']
>>> cut_lens = lens[lens, lens.age.cut(range(0, 81, 10), right=False, labels=labels).rename('年龄分组')]
```

取分组和年龄唯一的前10条数据来进行查看，如下所示：

```
>>> cut_lens['年龄分组', 'age'].distinct()[:10]
```

|　|年龄分组|age|
|:-|:---|:--|
|0|0-9|7|
|1|10-19|10|
|2|10-19|11|
|3|10-19|13|
|4|10-19|14|
|5|10-19|15|
|6|10-19|16|
|7|10-19|17|
|8|10-19|18|
|9|10-19|19|

对各个年龄分组下，用户的评分总数和评分均值进行查看，如下所示：

```
>>> cut_lens.groupby('年龄分组').agg(cut_lens.rating.count().rename('评分总数'), cut_lens.rating.mean().rename('评分均值'))
```

|　|年龄分组|评分均值|评分总数|
|:-|:---|:---|:---|
|0|0-9|3.767442|43|
|1|10-19|3.486126|8181|
|2|20-29|3.467333|39535|
|3|30-39|3.554444|25696|
|4|40-49|3.591772|15021|
|5|50-59|3.635800|8704|
|6|60-69|3.648875|2623|
|7|70-79|3.649746|197|

## Configuration {#section_bcy_gdc_wdb .section}

PyODPS提供了一系列的配置选项，可通过`odps.options`获得。可配置的MaxCompute选项，如下所示：

-   **通用配置**

    |选项|说明|默认值|
    |:-|:-|:--|
    |end\_point|MaxCompute Endpoint|None|
    |default\_project|默认Project|None|
    |log\_view\_host|LogView主机名|None|
    |log\_view\_hours|LogView保持时间（小时）|24|
    |local\_timezone|使用的时区，True表示本地时间，False表示UTC，也可用pytz 的时区|1|
    |lifecycle|所有表生命周期|None|
    |temp\_lifecycle|临时表生命周期|1|
    |biz\_id|用户ID|None|
    |verbose|是否打印日志|False|
    |verbose\_log|日志接收器|None|
    |chunk\_size|写入缓冲区大小|1496|
    |retry\_times|请求重试次数|4|
    |pool\_connections|缓存在连接池的连接数|10|
    |pool\_maxsize|连接池最大容量|10|
    |connect\_timeout|连接超时|5|
    |read\_timeout|读取超时|120|
    |completion\_size|对象补全列举条数限制|10|
    |notebook\_repr\_widget|使用交互式图表|True|
    |sql.settings|ODPS SQL运行全局hints|None|
    |sql.use\_odps2\_extension|启用MaxCompute2.0语言扩展|False|

-   **数据上传/下载配置**

    |选项|说明|默认值|
    |:-|:-|:--|
    |tunnel.endpoint|Tunnel Endpoint|None|
    |tunnel.use\_instance\_tunnel|使用Instance Tunnel获取执行结果|True|
    |tunnel.limited\_instance\_tunnel|限制Instance Tunnel获取结果的条数|True|
    |tunnel.string\_as\_binary|在string类型中使用bytes而非unicode|False|

-   **DataFrame配置**

    |选项|说明|默认值|
    |:-|:-|:--|
    |interactive|是否在交互式环境|根据检测值|
    |df.analyze|是否启用非ODPS内置函数|True|
    |df.optimize|是否开启DataFrame全部优化|True|
    |df.optimizes.pp|是否开启DataFrame谓词下推优化|True|
    |df.optimizes.cp|是否开启DataFrame列剪裁优化|True|
    |df.optimizes.tunnel|是否开启DataFrame使用tunnel优化执行|True|
    |df.quote|ODPS SQL后端是否用\`\`来标记字段和表名|True|
    |df.libraries|DataFrame运行使用的第三方库（资源名）|None|

-   **PyODPS ML配置**

    |选项|说明|默认值|
    |:-|:-|:--|
    |ml.xflow\_project|默认Xflow工程名|algo\_public|
    |ml.use\_model\_transfer|是否使用ModelTransfer获取模型PMML|True|
    |ml.model\_volume|在使用ModelTransfer时使用的Volume名称|pyodps\_volume|


