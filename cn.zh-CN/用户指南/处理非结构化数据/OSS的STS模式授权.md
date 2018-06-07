# OSS的STS模式授权 {#concept_ln3_c4v_ydb .concept}

创建外部表时Location访问OSS的账号支持传入明文AccessKeyId和AccessKeySecret，但这样做有泄露账号的风险。在某些场景下，这种风险是不可容忍的，因此MaxCompute提供了更为安全的方式来访问OSS。

MaxCompute结合了阿里云的访问控制服务（RAM）和令牌服务（STS）来解决账号的安全问题。您可通过以下两种方式授予权限：

-   当MaxCompute和OSS的owner是同一个账号时 ，可以直接在RAM控制台进行一键授权操作。
-   自定义授权。
    1.  首先需要在RAM中授权MaxCompute访问OSS的权限。创建角色，角色名如AliyunODPSDefaultRole或AliyunODPSRoleForOtherUser，并将策略内容设置为：

        ```
        --当MaxCompute和OSS的Owner是同一个账号
        {
        "Statement": [
        {
        "Action": "sts:AssumeRole",
        "Effect": "Allow",
        "Principal": {
        "Service": [
        "odps.aliyuncs.com"
        ]
        }
        }
        ],
        "Version": "1"
        }
        --当MaxCompute和OSS的Owner不是同一个账号
        {
        "Statement": [
        {
        "Action": "sts:AssumeRole",
        "Effect": "Allow",
        "Principal": {
        "Service": [
        "MaxCompute的Owner云账号id@odps.aliyuncs.com"
        ]
        }
        }
        ],
        "Version": "1"
        }
        ```

    2.  授予角色访问OSS必要的权限 AliyunODPSRolePolicy 。如下所示：

        ```
        {
        "Version": "1",
        "Statement": [
        {
        "Action": [
         "oss:ListBuckets",
         "oss:GetObject",
         "oss:ListObjects",
         "oss:PutObject",
         "oss:DeleteObject",
         "oss:AbortMultipartUpload",
         "oss:ListParts"
        ],
        "Resource": "*",
        "Effect": "Allow"
        }
        ]
        }
        --可自定义其他权限
        ```

    3.  再将权限`AliyunODPSRolePolicy`授权给该角色。

**说明：** 授权完成后，查看角色详情获取Role的Ran信息，后续创建OSS外部表时需指定这个Ran信息。

