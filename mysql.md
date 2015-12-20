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
基本过程

1. salve上的IO进程连接上master，并请求从指定日志文件的指定位置之后的日志内容
2. master接收到来自slave的IO进程的请求后，通过负责复制的IO进程根据请求信息读取指定位置之后的日志信息，返回给slave的IO进程。返回的信息中除了日志所包含的信息之外，还包括本次返回的信息已经到master端的bin-log文件的名称以及bin-log的位置
3. slave的IO进程接收到信息后，将接收到的日志内容依次添加到slave端的relay-log文件的末尾，并将读取到的master端的bin-log的文件名和位置记录到master-info文件中
4. slave的SQL进程检测到relay-log中新增加了内容后，会解析relay-log的内容成为在master端真实执行时候的那些可执行的内容，并在自身执行

######MHA架构
####备份恢复
####压力测试