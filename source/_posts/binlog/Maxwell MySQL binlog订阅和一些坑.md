---
title: Maxwell MySQL binlog订阅和一些坑
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/P60724-115835.jpg'
date: 2016-11-09 19:22:47
categories:
	- mysql
tags:
	- Maxwell
	- binlog
	- mysql
keywords:
	- Maxwell
    - Binlog
    - mysql
description:
---

## maxwell 相关资源

http://maxwells-daemon.io/
https://github.com/zendesk/maxwell
https://github.com/zendesk/open-replicator

## 配置MySQL master数据源

```
[mysqld]
server-id=1
log-bin=master
binlog_format=row
```

注意：
1. MySQL必须开启了binlogs，即log-bin指定了目录
2. binlog_format必须是row

## master数据源配置REPLICATION权限：

Maxwell需要储存他自己的一些状态数据，启动参数schema_database选型来指定，默认是`maxwell`.


```sql
GRANT ALL on maxwell.* to 'maxwell'@'%' identified by '123456';
GRANT SELECT, REPLICATION CLIENT, REPLICATION SLAVE on *.* to 'maxwell'@'%';

```


## 问题列表
1. 当binlog文件不存在时（被删除、移除、过期）
	- 无法启动maxwell
	- 正在运行的maxwell**`可能`**会stop

  
### 在阿里云RDS下的风险问题

1. binlog文件清理问题
2. binlog文件名在切换master主备或者阿里运维维护时会重置

RDS for MySQL 的 Binlog 生成和清理规则：

参考：[RDS for MySQL 之 Binlog 日志生成和清理规则](<https://help.aliyun.com/knowledge_detail/41815.html>)

### 其他问题

1. 阿里RDS的binlog在被复制完成后，会将之前的最后的binlog文件复制到其他地方，如果maxwell挂起时间较长，有可能未复制的binlog不能被复制过来。
	- 在AWS下的相关issue：https://github.com/zendesk/maxwell/issues/282
```
Amazon rds peridically deletes binlog files, `bin/maxwell` throws error and stops
```
	- Maxwell loses connection during RDS backups：https://github.com/zendesk/maxwell/issues/326
```
com.google.code.or.net.TransportException: Could not find first log file name in binary log index file
        at com.google.code.or.OpenReplicator.dumpBinlog(OpenReplicator.java:288)
```

### 当master中的binlog文件被删除后，无法启动

```java
09:35:06,785 ERROR MaxwellReplicator - Missing binlog 'mysql-bin.007213' on rdst5ai4d32fe3qd6if46.mysql.rds.aliyuncs.com
09:35:06,785 ERROR MaxwellReplicator - Transport exception #1236
com.google.code.or.net.TransportException: Could not find first log file name in binary log index file
	at com.google.code.or.OpenReplicator.dumpBinlog(OpenReplicator.java:343)
	at com.google.code.or.OpenReplicator.start(OpenReplicator.java:115)
	at com.zendesk.maxwell.replication.MaxwellReplicator.startReplicator(MaxwellReplicator.java:140)
	at com.zendesk.maxwell.replication.MaxwellReplicator.beforeStart(MaxwellReplicator.java:155)
	at com.zendesk.maxwell.util.RunLoopProcess.runLoop(RunLoopProcess.java:29)
	at com.zendesk.maxwell.Maxwell.start(Maxwell.java:160)
	at com.zendesk.maxwell.Maxwell.main(Maxwell.java:181)
09:35:06,786 INFO  Maxwell - starting shutdown


```