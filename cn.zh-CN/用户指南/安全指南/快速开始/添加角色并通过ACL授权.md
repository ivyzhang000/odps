# 添加角色并通过ACL授权 {#concept_kjy_2sz_vdb .concept}

场景描述：Jack是项目空间prj1的管理员，有三个新加入的项目组成员：Alice, Bob, Charlie，他们的角色是数据审查员。他们要申请如下权限：查看Table列表，提交作业，读取表userprofile。

对于这个场景的授权，项目空间管理员可以使用基于对象的 [ACL授权](https://www.alibabacloud.com/help/zh/doc-detail/27935.htm) 机制来完成。

操作方法：

```
    use prj1;
    add user aliyun$alice@aliyun.com; --添加用户
    add user aliyun$bob@aliyun.com;
    add user aliyun$charlie@aliyun.com;
    create role tableviewer; --创建角色
    grant List, CreateInstance on project prj1 to role tableviewer; --对角色赋权
    grant Describe, Select on table userprofile to role tableviewer;
    grant tableviewer to aliyun$alice@aliyun.com; --对用户赋予角色tableviewer
    grant tableviewer to aliyun$bob@aliyun.com;
    grant tableviewer to aliyun$charlie@aliyun.com;
```

