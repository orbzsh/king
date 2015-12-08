####参考
[慢查询监控器](https://github.com/box/Anemometer)

####参数
innodb_flush_log_at_trx_commit=2

    该属性主要为数据库的ACID原则进行服务的，默认为1
    1的时候缓存会在事务提交时或者每秒钟时进行磁盘的刷新操作
    2的时候缓存会在提交事务时写入到事务日志但不会刷新磁盘，然后在每秒钟进行磁盘刷新操作

innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=1

    InnoDB使用buffer pool来缓存索引和行数据，值越大，磁盘I/O就越少
    最大设置可到物理内存的80%

binlog-format=MIXED
####主从同步
######MHA架构
####备份恢复
####压力测试