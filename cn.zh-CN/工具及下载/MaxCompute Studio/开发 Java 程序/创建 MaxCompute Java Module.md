# 创建 MaxCompute Java Module {#concept_twh_wxf_vdb .concept}

MaxCompute Studio能支持用户开发Java UDF和MR，首先需要新建一个MaxCompute Java Module。

## 创建module {#section_l54_3yf_vdb .section}

依次点击 `File | new | module` module类型为`MaxCompute Java`，配置`Java JDK`。点击**next**，输入module名，点击**finish**。studio会帮用户自动创建一个maven module，并引入MaxCompute相关依赖，具体请查看pom文件。

## module结构说明 {#section_m54_3yf_vdb .section}

至此，一个能开发MaxCompute Java程序的module已建立，如下图的mDev。主要目录包括:

-   src/main/java：用户开发java程序源码。

-   examples：示例代码，包括单测示例，用户可参考这里的例子开发或编写UT。

-   warehouse：本地运行时需要的schema和data。


![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12129/1928_zh-CN.png)

