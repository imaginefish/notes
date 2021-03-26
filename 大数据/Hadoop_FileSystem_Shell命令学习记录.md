# Hadoop FileSystem Shell命令学习记录

## 总览

FileSystem（FS）shell 包含了与 Hadoop 分布式系统文件（HDFS）和其他 Hadoop 支持的文件系统，如本地文件系统（Local FS）、WebHDFS、S3 FS 等直接交互的多种类shell命令。

```shell
bin/hadoop fs <args>
```

所有的 FS shell 命令都会使用路径的 URIs 作为参数。URI 格式为`scheme://authority/path`。对于 HDFS，schema为`hdfs`,对于本地文件系统schema为`file`。schema 和 authority 是可选的。如果不指定，默认的 schema 会在被使用的配置文件中指定。一个 HDFS 文件或者目录，例如 /parent/child 可以被指定为`hdfs://namenodehost/parent/child`

