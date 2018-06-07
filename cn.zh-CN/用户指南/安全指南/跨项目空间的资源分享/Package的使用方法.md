# Package的使用方法 {#concept_vj4_rk1_wdb .concept}

Package的使用涉及到两个主体：Package创建者和Package使用者。

-   Package创建者项目空间是资源提供方，它将需要分享的资源及其访问权限进行打包，然后许可Package使用方来安装使用。
-   Package使用者项目空间是资源使用方，它在安装资源提供方发布的Package之后，便可以直接跨项目空间访问资源。

下面分别介绍Package创建者和Package使用者所涉及的操作。

## Package创建者 {#section_sbs_1l1_wdb .section}

-   **创建Package**

    ```
    create package ;
    ```

    **说明：** 

    -   仅project的owner有权限进行该操作。
    -   目前创建的package名称不能超过128个字符。
-   **将分享的资源添加到Package**

    ```
        add project_object to package package_name [with privileges privileges];--将对象添加到Package
        remove project_object from package package_name;--将对象从Package移除
        project_object ::= table table_name |
                       instance inst_name |
                       function func_name |
                       resource res_name
        privileges ::= action_item1, action_item2, ...
    ```

    **说明：** 

    -   目前支持的对象类型不包括Project类型，也就是不允许通过Package在其他Project中创对象。
    -   添加到Package中的不仅仅是对象本身，还包括相应的操作权限。当没有通过\[with privileges privileges\]来指定操作权限时，默认为只读权限，即Read/Describe/Select。“对象及其权限”被看作一个整体，添加后不可被更新。若有需要，只能删除和重新添加。
-   **许可其他项目空间使用Package**

    ```
    allow project <prjname> to install package <pkgname> [using label <number>];
    ```

-   **撤销其他项目空间使用Package的许可**

    ```
    disallow project <prjname> to install package <pkgname>;
    ```

-   **删除Package**

    ```
    delete package <pkgname>;
    ```

-   **查看已创建和已安装的Package列表**

    ```
    show packages;
    ```

-   **查看Package详细信息**

    ```
    describe package <pkgname>;
    ```


## Package使用者 {#section_kwk_fl1_wdb .section}

-   **安装Package**

    ```
    install package <pkgname>;
    ```

    对于安装Package来说，要求pkgName的格式为：`<projectName>.<packageName>`。

    **说明：** 仅project的Owner有权限进行该操作。

-   **卸载Package**

    ```
    uninstall package <pkgname>;
    ```

    对于卸载Package来说，要求pkgName的格式为：`<projectName>.<packageName>`

-   **查看Package**

    ```
        show packages;
        --查看已创建和已安装的package列表
        describe package <pkgname>;
        --查看package详细信息
    
    ```

-   **使用方项目给本项目其他成员授权访问Package**

    被安装的Package是一种独立的MaxCompute对象类型。若要访问Package里的资源\(即其他项目空间分享给您的资源\)，您必须拥有对该Package的Read权限。

    如果请求者没有Read权限，则需要向ProjectOwner或Admin申请。ProjectOwner或Admin可以通过ACL授权机制来完成。

    比如，如下的ACL授权允许云账号用户LIYUN$odps\_test@aliyun.com访问Package里的资源：

    ```
        use prj2;
        install package prj1.testpkg;
        grant read on package prj1.testpackage to user aliyun$odps_test@aliyun.com;
    ```


## 场景示例 {#section_hx1_3l1_wdb .section}

场景描述：Jack是项目空间prj1的管理员。John是项目空间prj2的管理员。由于业务需要， Jack希望将其项目空间prj1中的某些资源\(比如datamining.jar及sampletable表\)分享给John的项目空间prj2。 如果项目空间prj2的用户Bob需要访问这些资源，那么prj2管理员可以通过ACL自主授权，无需Jack参与。

操作步骤：

1.  项目空间prj1的管理员Jack在项目空间prj1中创建资源包\(package\)。

    ```
        use prj1;
        create package datamining; --创建一个package
        add resource datamining.jar to package datamining; --添加资源到package
        add table sampletable to package datamining; --添加table到package
        allow project prj2 to install package datamining; --将package分享给项目空间prj2
    ```

2.  项目空间prj2管理员Bob在项目空间prj2中安装package。

    ```
        use prj2;
        install package prj1.datamining; --安装一个package
        describe package prj1.datamining; --查看package中的资源列表
    ```

3.  Bob对package进行自主授权。

    ```
        use prj2;
        grant Read on package prj1.datamining to user aliyun$bob@aliyun.com; --通过ACL授权Bob使用package
    ```


