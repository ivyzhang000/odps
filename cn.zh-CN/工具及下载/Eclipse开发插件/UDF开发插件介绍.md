# UDF开发插件介绍 {#concept_vz1_scd_wdb .concept}

在本章节我们将介绍如何使用Eclipse插件开发并在本地运行UDF。UDAF、UDTF的编写执行过程与UDF类似，均可参考UDF的示例介绍完成。ODPS Eclipse插件提供两种运行UDF的方式：菜单栏和右键单击快速运行方式。

## 菜单栏运行 {#section_t43_ckx_wdb .section}

1.  从菜单栏选择**Run** \> **Run Configurations…**弹出如下对话框：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3192_zh-CN.PNG)

2.  用户可以新建一个Run Configuration，选择运行的UDF类及类型、选择ODPS Project、填写输入表信息，如：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3193_zh-CN.JPG)

    上述配置中，Table表示UDF的输入表，Partitions表示读取某个分区下的数据，分区由逗号分隔，Columns表示列，将依次作为UDF函数的参数被传入，列名由逗号分隔。

3.  点击**Run**运行，运行结果将显示在控制台中：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3194_zh-CN.JPG)


## 右键单击快速运行 {#section_ap3_ckx_wdb .section}

1.  选中一个udf.java文件（比如：UDFExample.java）并单击鼠标右键，选择**Run As** \> **Run UDF|UDAF|UDTF**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3196_zh-CN.png)

2.  配置信息如下：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3197_zh-CN.png)

    上述配置中，Table表示UDF的输入表，Partitions表示读取某个分区下的数据，分区由逗号分隔，Columns表示列，将依次作为UDF函数的参数被传入，列名由逗号分隔。

3.  点击**Finish**后，运行UDF，获得输出结果。

## 运行用户自定义UDF程序 {#section_gp3_ckx_wdb .section}

右击一个工程并选择**New** \> **UDF**（或者选择菜单栏**File** \> **New** \> **UDF**\)。

填写UDF类名然后点击**Finish**。在对应的src目录下生成与UDF类名同名的Java文件，编辑该java文件内容：

```
package odps;
import com.aliyun.odps.udf.UDF;
public class UserUDF extends UDF {
      /**
       * project: example_project 
       * table: wc_in1 
       * columns: col1,col2
       * 
       */
      public String evaluate(String a, String b) {
        return "ss2s:" + a + "," + b;
      }
}
```

右击该java文件（如UserUDF.java），选择**Run As**，再选择**ODPS UDF|UDTF|UDAF**:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3200_zh-CN.png)

配置如下对话框：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12153/3201_zh-CN.png)

点击**Finish**，得出结果：

```
ss2s:A1,A2
ss2s:A1,A2
ss2s:A1,A2
ss2s:A1,A2
```

本示例中仅给出UDF的运行示例，UDTF的运行方式与UDF基本相同，不做特殊说明。

