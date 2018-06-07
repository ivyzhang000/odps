# BufferedWriter上传示例 {#concept_npv_cpg_vdb .concept}

```
// 初始化 MaxCompute 和 tunnel 的代码
RecordWriter writer = null;
TableTunnel.UploadSession uploadSession = tunnel.createUploadSession(projectName, tableName);
try {
  int i = 0;
  // 生成TunnelBufferedWriter的实例
  writer = uploadSession.openBufferedWriter();
  Record product = uploadSession.newRecord();
  for (String item : items) {
    product.setString("name", item);
    product.setBigint("id", i);
    // 调用write接口写入数据
    writer.write(product);
    i += 1;
  }
} finally {
  if (writer != null) {
    // 关闭TunnelBufferedWriter
    writer.close();
  }
}
// uploadSession提交，结束上传
uploadSession.commit();
```

