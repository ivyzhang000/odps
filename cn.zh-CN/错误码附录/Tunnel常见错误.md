# Tunnel常见错误 {#concept_ccs_p2w_tdb .concept}

Tunnel的异常信息通常如下所示：

```
requestId=request_id, ErrorCode=InvalidArgument, ErrorMessage=Invalid argument
```

其中，requestId是标志用户请求的唯一标志，ErrorCode是表明错误类型，ErrorMessage是详细异常信息。

常见的异常信息说明如下：

|异常类型|异常信息|补充说明|
|:---|:---|:---|
|AccessDenied|Access Denied|权限不足。|
|CorruptedDataStream|The data stream was corrupted, please try again later| |
|DataUnderReplication|The specified table data is under replication and you cannot initiate upload or download at this time. Please try again later|数据处于跨级群复制状态，无法操作。|
|DataVersionConflict|The specified table has been modified since the upload or download initiated and table data is being replicated at this time. Please initiate another download or upload later|当前集群上的数据处于复制状态，无法进行数据操作。请稍后重试。|
|DataStoreError|存储错误|建议联系管理员|
|FlowExceeded|Your flow quota is exceeded|数据长传/下载超过流量限制。建议检查控制并发量，若确实需要增加并发，请联系Project owner或管理员评估流量压力。|
|InConsistentBlockList|The specified block list is not consistent with the uploaded block list on server side| |
|IncompleteBody|You did not provide the number of bytes specified by the Content-Length HTTP header| |
|InternalServerError|Service internal error, please try again later|服务内部错误，建议重试或联系管理员。|
|InvalidArgument|Invalid argument|参数不合法。|
|InvalidBlockID|The specified block id is not valid|非法BlockID。|
|InvalidColumnSpec|The specified columns is not valid|非法列名， 通常是指定列下载时列名错误。|
|InvalidRowRange|The specified row range is not valid|指定行数不合法，通常是超出了数据最大size或者为0，建议检查相关参数。|
|InvalidStatusChange|You cannot change the specified upload or download status| |
|InvalidResourceSpec|通常为Project/Table/Partition信息与Session不一致|建议检查相关信息并重试。|
|InvalidURI|Couldn’t parse the specified URI| |
|InvalidUriSpec|The specified uri spec is not valid| |
|InvalidProjectTable|项目或表名称不合法|建议检查相关名称。|
|InvalidPartitionSpec|分区信息不合法|请检查分区信息，正确值应该如：pt=’1’,ct=’2016’。|
|MalformedDataStream|The data stream you provided was not well-formed or did not validate against schema|数据格式错误，通常是网络断开或Schema与Table的不一样导致。|
|MalformedHeaderValue|An HTTP header value was malformed| |
|MalformedXML|The XML you provided was not well-formed or did not validate against schema| |
|MaxMessageLengthExceeded|Your request was too big| |
|MethodNotAllowed|The specified method is not allowed against this resource|方法不支持，通常是尝试导出View。目前不支持View导出。|
|MissingContentLength|You must provide the Content-Length HTTP header| |
|MissingPartitionSpec|You need to specify a partitionspec along with the specified table|缺失分区信息，分区表操作必须携带分区信息，建议补充分区信息。|
|MissingRequestBodyError|The request body is missing| |
|MissingRequiredHeaderError|Your request was missing a required header| |
|NoPermission|You do not have enough privilege to complete the specified operation| |
|NoSuchData|The uploaded data within this uploaded no longer exists| |
|NoSuchPartition|The specified partition does not exist|指定的分区不存在，Tunnel不会创建分区，需要您自行创建分区后再上传下载。|
|NoSuchUpload|The specified upload id does not exist|指定的upload id指定的不存在。|
|NoSuchDownload|The specified download id does not exist|指定的download id不存在。|
|NoSuchProject|The specified project name does not exist|项目不存在，建议检查相关名称。|
|NoSuchTable|The specified table name does not exist|表不存在，建议检查相关名称。|
|NoSuchVolume|The specified volume name does not exist|指定的volume name不存在。|
|NoSuchVolumeFile|The specified volume file does not exist|指定的volume file指定的不存在。|
|NoSuchVolumePartition|The specified volume partition does not exist|指定的volume partition不存在。|
|NoPermission|权限不足通常原因是无相关权限或者设置了IP白名单|建议检查权限是否正确。|
|NotImplemented|A header you provided implies functionality that is not implemented| |
|ObjectModified|The specified object has been modified since the specified timestamp| |
|RequestTimeOut|Your socket connection to the server was not read from or written to within the timeout period| |
|ServiceUnavailable|Service is temporarily unavailable, Please try again later| |
|StatusConflict|You cannot complete the specified operation under the current upload or download status|Session过期或者已经Commit过，重新创建Session上传。|
|TableModified|The specified table has been modified since the download initiated. Try initiate another download|表数据在上传下载期间被其他任务改动，建议重新创建Session重试。|
|Unauthorized|The request authorization header is invalid or missing|通常是AccessId或AccessKey有误，或本地机器时间与服务端时间差距15分钟以上。|
|UnexpectedContent|This request does not support content| |

