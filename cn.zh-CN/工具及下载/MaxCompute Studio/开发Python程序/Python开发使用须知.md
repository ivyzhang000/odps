# Python开发使用须知 {#concept_vt2_sdd_xdb .concept}

MaxCompute Studio支持您在Intellij IDEA中完成Python相关的开发，包括UDF和Pyodps脚本，但使用前必须安装**Python**、**Pyodps**和**IDEA的Python插件**。

## 安装Pyodps {#section_ngm_b3d_xdb .section}

Pyodps是MaxCompute的Pyodps SDK，详情请参见[Python SDK](../cn.zh-CN/用户指南/SDK/Python SDK.md)。

## 安装Python插件 {#section_ogm_b3d_xdb .section}

在Intellij IDEA的插件仓库中搜索**Python Community Edition**插件并安装。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13746/3389_zh-CN.png)

## 配置Python依赖 {#section_ycn_z3d_xdb .section}

配置Studio Module对Python的依赖，即可进行MaxCompute Python的开发。

1.  导航至**File** \> **Project structure**，添加Python SDK。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13746/3391_zh-CN.png)

2.  导航至**File** \> **Project structure**，添加Python Facets。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13746/3392_zh-CN.png)

3.  导航至**File** \> **Project structure**，配置Module依赖Python Facets。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13746/3393_zh-CN.png)


