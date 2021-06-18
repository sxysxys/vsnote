## sql查询语句的执行

![img](https://static001.geekbang.org/resource/image/0d/d9/0d2070e8f84c4801adbfa03bda1f98d9.png)

- 连接器：保持与客户端的长连接。
- 查询缓存：记录之前的sql查询结果，如果下次查询命中缓存，就返回相应结果。
- 分析器：对送入的sql进行语句的分析，检测语法错误，判断它要做什么。
- 优化器：对sql的索引、连接顺序等进行一个优化。
- 执行器：将sql送入存储引擎接口，由存储引擎进行执行sql并返回结果。

## sql更新语句的执行

整体流程和上面一样，但是多了两个日志模块。

> 引擎innoDB特有日志：redo log

- 是引擎层的日志，执行器将sql送入后进行记录。
- 用来记录更新语句的，更新语句不会立即执行（存入磁盘），而是先更新内存，再通过redo log记录下来，再系统不繁忙的时候执行相应的更新磁盘操作。

![img](https://static001.geekbang.org/resource/image/16/a7/16a7950217b3f0f4ed02db5db59562a7.png)

write pos 和 check point之间的数据就是还能写的数据区域，其余的区域就是此时还未执行的更新语句，如果write pos追上了check point，数据库此时必须停下来将check point往前推进，执行一些更新语句。

- 可以实现数据库断开后的异常恢复，通过这种机制保证更新数据不丢失。

> Server层日志：binlog归档日志

1. redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
2. redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

> 更新语句执行流程

![2e5bff4910ec189fe1ee6e2ecc7b4bbe](mysql架构.assets/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png)

> 数据库备份的恢复

1. 定期进行数据库的备份
2. 主要使用的就是binlog，拿到备份时间点到当前的binlog，执行恢复操作。

> 总结

- redo log主要是保证了数据库的crash-safe能力，其实刷脏页什么的和read log也没关系，主要它是用来记录的，防止崩溃后丢失更新。
- binlog记录操作，用于恢复。

## 事务隔离

事务支持：引擎层

> 事务隔离的几种级别

> 注意事项

- 少使用长事务（回滚日志记录信息，无法释放）

## 索引模型

- 覆盖索引
- 前缀索引
- 索引下推

























































































































