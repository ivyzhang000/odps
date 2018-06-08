# 创建 MaxCompute Script Module {#concept_kcs_ll2_vdb .concept}

开发 MaxCompute Script 前，需要创建一个 MaxCompute Script Module，创建时存在以下两种情况：

## 本地没有 script 文件 {#section_ens_sl2_vdb .section}

本地没有 script 文件时，可通过 Intellij 创建一个全新的 Module。

**操作步骤**

1.  打开或者新建一个 MaxCompute Studio Project。本文以新建为例，单击菜单栏中的**File** ，导航至**New** \> **Project** 。如下图所示：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1781_zh-CN.png)

2.  选择左侧导航栏中的**MaxCompute Studio** ，单击**Next** 。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1782_zh-CN.png)

3.  填写 Project Name，单击 **Finish**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1783_zh-CN.png)

    **说明：** 

    如果之前有打开的 Project，将会提示您是否在当前窗口中打开（即关掉之前的 Project），选择 **This Window**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1784_zh-CN.png)

4.  创建完成后，页面如下图所示，您即可在此项目中开发 SQL 脚本。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1785_zh-CN.png)


## 本地已有 script 文件 {#section_lns_sl2_vdb .section}

假如本地某个文件夹下已经有很多 script，此时需要用 MaxCompute studio 来编辑这些 script，您可直接打开一个 Module。

**操作步骤**

1.  在**scripts** 文件夹下创建一个 MaxCompute 的连接配置文件 [odps\_config.ini](../cn.zh-CN/准备工作/安装并配置客户端.md)，在里面配置与 MaxCompute 连接的鉴权信息：

    -   project\_name=xxxxxxxx
    -   access\_id=xxxxxxxxxx
    -   access\_key=xxxxxxxxx
    -   end\_point=xxxxxxxxx
    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1786_zh-CN.png)

2.  打开 IDEA，导航至 **File** \> **open**， 选择您的**scripts** 文件夹。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12125/1787_zh-CN.png)

3.  MaxCompute Studio 会探测该文件夹下是否存在 odps\_config.ini 文件，根据这个文件里的配置信息抓取服务端的 meta，然后编译文件夹下的所有 script。


