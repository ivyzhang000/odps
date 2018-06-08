# MaxCompute术语表 {#concept_wxh_rxv_tdb .concept}

## A {#section_utx_txv_tdb .section}

-   **AccessKey**

    AccessKey（简称AK，包括Access Key Id和Access Key Secret），是访问阿里云API的密钥，在阿里云官网注册云账号后，可在[accesskeys 管理](https://ak-console.aliyun.com/#/)页面生成，用于标识用户，为访问MaxCompute或者其他云产品做签名验证。**Access Key Secret必须保密。**

-   **安全**

    MaxCompute多租户数据安全体系，主要包括：用户认证、项目空间的用户与授权管理、跨项目空间的资源分享以及项目空间的数据保护。关于MaxCompute安全操作的更多详情请参见[安全指南](../cn.zh-CN/用户指南/安全指南/目标用户.md)。


## B {#section_nkb_vxv_tdb .section}

**Table （表）**

表是MaxCompute的数据存储单元，详情请参见[基本概念\>表](cn.zh-CN/产品简介/基本概念/表.md)。

**Console**

MaxCompute运行在Window/Linux下的客户端工具，通过Console可以提交命令完成Project管理、DDL、DML等操作。对应的工具安装和常用参数请参见[客户端](../cn.zh-CN/工具及下载/客户端.md)。

## D {#section_o1d_vxv_tdb .section}

-   **Data type**

    MaxCompute表中所有列对应的数据类型。目前支持的数据类型详情请参见[基本概念\>数据类型](cn.zh-CN/产品简介/基本概念/数据类型.md)。

-   **DDL**

    Data Definition Language（数据定义语言）。比如创建表、创建视图等操作，MaxCompute DDL语法请参见[用户指南\>DDL 语句](../cn.zh-CN/用户指南/SQL/DDL语句.md)。

-   **DML**

    Data Manipulation Language（数据操作语言）。比如INSERT操作，MaxCompute DML语法请参见 [用户指南\>INSERT 操作](../cn.zh-CN/用户指南/SQL/INSERT操作.md)。


-   **Partition（分区）**

    分区Partition是指一张表下，根据分区字段（一个或多个组合）对数据存储进行划分。也就是说，如果表没有分区，数据是直接放在表所在的目录下。如果表有分区，每个分区对应表下的一个目录，数据是分别存储在不同的分区目录下。关于分区的更多介绍请参见[基本概念\>分区](cn.zh-CN/产品简介/基本概念/分区.md)。

-   **fuxi**

    伏羲（fuxi）是飞天平台内核中负责资源管理和任务调度的模块，同时也为应用开发提供了一套编程基础框架。MaxCompute底层任务调度模块即fuxi的调度模块。


**Role（角色）**

角色是MaxCompute安全功能里使用的概念，可以看成是拥有相同权限的用户的集合。多个用户可以同时存在于一个角色下，一个用户也可以隶属于多个角色。给角色授权后，该角色下的所有用户拥有相同的权限。关于角色管理的更多介绍请参见[安全指南\>角色管理](../cn.zh-CN/用户指南/安全指南/角色管理.md)。

## M {#section_f2f_vxv_tdb .section}

**MapReduce**

MaxCompute处理数据的一种编程模型，通常用于大规模数据集的并行运算。您可以使用MapReduce提供的接口（Java API）编写MapReduce程序，来处理MaxCompute中的数据。编程思想是将数据的处理方式分为**Map（映射）**和**Reduce（规约）**。

在正式执行Map前，需要将输入的数据进行**分片**。所谓分片，就是将输入数据切分为大小相等的数据块，每一块作为单个Map Worker的输入被处理，以便于多个Map Worker同时工作。每个Map Worker在读入各自的数据后，进行计算处理，最终通过Reduce函数整合中间结果，从而得到最终计算结果。详情请参见 [MapReduce](../cn.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md)。

## O {#section_xwf_vxv_tdb .section}

**ODPS**

ODPS是MaxCompute的原名。

## P {#section_wpg_vxv_tdb .section}

**Project（项目）**

项目空间（Project）是MaxCompute的基本组织单元，它类似于传统数据库的Database或Scheme的概念，是进行多用户隔离和访问控制的主要边界。详情请参见[基本概念\>项目空间](cn.zh-CN/产品简介/基本概念/项目空间.md)。

## S {#section_n3h_vxv_tdb .section}

-   **SDK**

    Software Development Kits软件开发工具包。一般都是一些被软件工程师用于为特定的软件包、软件实例、软件框架、硬件平台、操作系统、文档包等建立应用软件的开发工具的集合。MaxCompute目前支持 [JAVA SDK](../cn.zh-CN/用户指南/MapReduce/Java SDK.md)和[Python SDK](../cn.zh-CN/用户指南/SDK/Python SDK.md)。

-   **授权**

    项目空间管理员或者project owner授予您对MaxCompute中的Object（或称之为客体，例如表，任务，资源等）进行某种操作的权限，包括读、写、查看等。授权的具体操作请参见[安全指南 \> 用户管理](../cn.zh-CN/用户指南/安全指南/用户管理.md)。

-   **沙箱**

    MaxCompute MapReduce及UDF程序在分布式环境中运行时受到[Java沙箱](../cn.zh-CN/用户指南/MapReduce/Java沙箱.md)的限制。

-   **Instance（实例）**

    作业的一个具体实例，表示实际运行的Job，类同Hadoop中Job的概念。详情请参见[基本概念\>任务实例](cn.zh-CN/产品简介/基本概念/任务实例.md)。


## T {#section_ib3_vxv_tdb .section}

**Tunnel**

MaxCompute的数据通道，提供高并发的离线数据上传下载服务。您可以使用Tunnel服务向MaxCompute批量上传数据或者将数据下载。相关命令请参见[Tunnel命令操作](../cn.zh-CN/用户指南/数据上传下载/Tunnel命令操作.md)或[批量数据通道SDK](../cn.zh-CN/用户指南/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md)。

## U {#section_e53_vxv_tdb .section}

-   **UDF**

    广义的UDF，即User Defined Function，MaxCompute提供的Java编程接口开发自定义函数，详情请参见[用户指南\>UDF](../cn.zh-CN/用户指南/SQL/UDF/UDF概述.md)。

    狭义的UDF指用户自定义标量值函数（User Defined Scalar Function），它的输入与输出是一对一的关系，即读入一行数据，写出一条输出值。

-   **UDAF**

    User Defined Aggregation Function，自定义聚合函数，它的输入与输出是多对一的关系， 即将多条输入记录聚合成一条输出值。可以与SQL中的Group By语句联用。详情请参见[Java UDF\>UDAF](../cn.zh-CN/用户指南/SQL/UDF/Java UDF.md)。

-   **UDTF**

    User Defined Table Valued Function，自定义表值函数，用来解决一次函数调用输出多行数据的场景，也是唯一能返回多个字段的自定义函数。而UDF只能一次计算输出一条返回值。详情请参见[Java UDF\>UDAF](../cn.zh-CN/用户指南/SQL/UDF/Java UDF.md)。


## Z {#section_tnj_vxv_tdb .section}

**Resource（资源）**

资源（Resource）是MaxCompute中特有的概念。您如果想使用MaxCompute的自定义函数（UDF）或MapReduce功能，都需要依赖资源来完成。详情请参见[基本概念\>资源](cn.zh-CN/产品简介/基本概念/资源.md)。

