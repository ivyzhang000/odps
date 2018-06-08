# JAVA UDF开发 {#concept_edn_3yy_5db .concept}

MaxCompute的UDF包括UDF、UDAF和UDTF三种函数。通常情况下，这三种函数被统称为UDF。

实现JAVA UDF使用Maven的用户可以从[Maven库](http://search.maven.org/)中搜索odps-sdk-udf获取不同版本的Java SDK，相关配置信息如下所示：

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
    <version>0.20.7</version>
</dependency>
```

通常情况下，JAVA UDF的开发可以通过以下几种方式：

-   使用[MaxCompute Studio完成JAVA UDF开发整个流程](../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/开发和调试UDF.md)。
-   使用[Eclipse插件开发和调试JAVA UDF](../cn.zh-CN/工具及下载/Eclipse开发插件/UDF开发插件介绍.md)，导出Jar包，然后通过命令或者DataWorks[添加资源](../cn.zh-CN/用户指南/常用命令/资源操作.md)后再[注册函数](../cn.zh-CN/用户指南/常用命令/函数操作.md)。

本文中会分别给出UDF、UDAF、UDTF的代码示例，并通过两种方式给出开发UDF完整流程步骤示例（UDAF、UDTF操作步骤与UDF操作步骤一样）。

**说明：** 

-   关于自定义函数注册和注销、查看函数列表的相关命令语句请参见[函数操作](../cn.zh-CN/用户指南/常用命令/函数操作.md)。
-   Java和MaxCompute的数据类型对应关系，请参见[参数与返回值类型](../cn.zh-CN/用户指南/SQL/UDF/Java UDF.md)。

## UDF示例 {#section_q5g_1zy_5db .section}

下面将为您介绍一个字符小写转换功能的UDF实现示例。

-   **使用MaxCompute Studio开发**
    1.  **准备工具环境**。

        这里假设已经完成环境准备，包括[安装Studio](../cn.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/安装IntelliJ IDEA.md)并在Studio上[创建MaxCompute项目链接](../cn.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md)以及[创建MaxCompute Java Module](../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/创建 MaxCompute Java Module.md)。

    2.  **编写代码**。

        在配置好的Java Module下创建Java文件。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/1573_zh-CN.png)

        直接选择MaxCompute java然后name里输入`package名称.文件名`，Kind选择UDF。 然后编辑如下代码：

        ```
        package <package名称>;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
        public String evaluate(String s) {
         if (s == null) { return null; }
         return s.toLowerCase();
        }
        }
        ```

        **说明：** 若需本地调试java udf，请参见[开发和调试UDF](../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/开发和调试UDF.md)

    3.  **注册MaxCompute UDF**。

        如下图所示，右键单击UDF的Java文件，选择**Deploy to server**，弹框里选择注册到那个MaxCompute project，输入`function name`，Resource name也可以修改。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/1574_zh-CN.png)

        填写好后，单击**OK**即可。注册成功后会有提示。

    4.  **试用UDF**。

        打开SQL脚本，执行代码如`select Lower_test(‘ABC’);`结果如下图所示：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/1575_zh-CN.png)

        **说明：** Studio中编写SQL脚本请参见[编写SQL脚本](../cn.zh-CN/工具及下载/MaxCompute Studio/开发 SQL 程序/编写SQL脚本.md)。

-   **使用Eclipse插件开发**
    1.  **创建工程**

        此处假设已经在Eclipse插件创建好一个MaxCompute（原名ODPS）工程，详情请参见[创建MaxCompute工程](../cn.zh-CN/工具及下载/Eclipse开发插件/创建 MaxCompute 工程.md)。

    2.  **编写代码**

        按照MaxCompute UDF框架的规定，实现函数功能，并进行编译。示例如下：

        ```
        
        package org.alidata.odps.udf.examples;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
        public String evaluate(String s) {
        if (s == null) { return null; }
        return s.toLowerCase();
        }
        }
        ```

        将这个Jar包命名为**my\_lower.jar**。

        **说明：** 

        -   更详细的开发调试代码的介绍请参见[UDF开发插件介绍](../cn.zh-CN/工具及下载/Eclipse开发插件/UDF开发插件介绍.md)。
        -   SDK的使用信息请参见[UDF SDK](../cn.zh-CN/用户指南/SQL/UDF/UDF概述.md) 。
    3.  **添加资源**

        在运行UDF之前，必须指定引用的UDF代码。代码通过资源的形式添加到MaxCompute中。Java UDF必须被打成Jar包，以Jar资源添加到MaxCompute中，UDF框架会自动加载Jar包，运行用户自定义的UDF。

        **说明：** MaxCompute MapReduce也用到了资源这一特有概念，[MapReduce](../cn.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md)文档中对资源的使用也有阐述。

        执行如下命令：

        ```
        
        add jar my_lower.jar;
        -- 如果存在同名的资源请将这个jar包重命名
        -- 并注意修改下面示例命令中相关jar包的名字
        -- 又或者直接使用-f选项覆盖原有的jar资源
        ```

    4.  **注册UDF函数**

        Jar包被上传后，使得MaxCompute有条件自动获取代码并运行。但此时仍然无法使用这个UDF，因为MaxCompute中并没有关于这个UDF的任何信息。因此需要在MaxCompute中注册一个唯一的函数名，并指定这个函数名与哪个jar资源的哪个类对应。

        执行如下命令：

        ```
        CREATE FUNCTION test_lower AS org.alidata.odps.udf.examples.Lower USING my_lower.jar;
        ```

    5.  在SQL中使用此函数进行验证：

        ```
        select test_lower('A') from my_test_table;
        ```


## UDAF示例 {#section_zvj_3bz_5db .section}

UDAF的注册方式与UDF基本相同，使用方式与内建函数中的[聚合函数](../cn.zh-CN/用户指南/SQL/内建函数/聚合函数.md)相同。计算平均值的UDAF的代码示例，如下所示：

```

package org.alidata.odps.udf.examples;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.io.Writable;
import com.aliyun.odps.udf.Aggregator;
import com.aliyun.odps.udf.UDFException;
/**
* project: example_project
* table: wc_in2
* partitions: p2=1,p1=2
* columns: colc,colb,cola
*/
public class UDAFExample extends Aggregator {
@Override
public void iterate(Writable arg0, Writable[] arg1) throws UDFException {
LongWritable result = (LongWritable) arg0;
for (Writable item : arg1) {
Text txt = (Text) item;
result.set(result.get() + txt.getLength());
}
}
@Override
public void merge(Writable arg0, Writable arg1) throws UDFException {
LongWritable result = (LongWritable) arg0;
LongWritable partial = (LongWritable) arg1;
result.set(result.get() + partial.get());
}
@Override
public Writable newBuffer() {
return new LongWritable(0L);
}
@Override
public Writable terminate(Writable arg0) throws UDFException {
return arg0;
}
}
```

## UDTF示例 {#section_a1j_lbz_5db .section}

UDTF的注册和使用方式与UDF相同。代码示例如下：

```

package org.alidata.odps.udtf.examples;
import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.UDTFCollector;
import com.aliyun.odps.udf.annotation.Resolve;
import com.aliyun.odps.udf.UDFException;
// TODO define input and output types, e.g., "string,string->string,bigint".
@Resolve({"string,bigint->string,bigint"})
public class MyUDTF extends UDTF {
@Override
public void process(Object[] args) throws UDFException {
String a = (String) args[0];
Long b = (Long) args[1];
for (String t: a.split("\\s+")) {
forward(t, b);
}
}
}
```

MaxCompute提供了很多内建函数来满足您的计算需求，同时您还可以通过创建自定义函数来满足不同的计算需求。详情请参见[创建自定义函数](https://help.aliyun.com/document_detail/30270.html)。

