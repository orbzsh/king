####redis常用命令
####redis持久化机制
rdb
```
遍历redis所有数据库中的键值对，将未过期的数据写入到RDB磁盘文件中
save命令会阻塞redis服务器进程，直到RDB文件创建完毕
bgsave命令则会开启一个子进程，直到RDB文件创建完毕，不会影响正常的服务
```
```
save seconds changes 
指出在多长时间内，有多少次更新操作，就将数据同步到数据文件 rdb 。 
比如默认配置文件中的设置，就设置了三个条件 
save 900 1 // 900 秒内至少有 1 个 key 被改变 
save 300 10 // 300 秒内至少有 300 个 key 被改变 
save 60 10000 // 60 秒内至少有 10000 个 key 被改变
```
aof
```
通过保存redis服务器执行的写命令来记录数据库状态
当redis执行一个写、删除命令时，同时会将该命令写入到一个aof文件
当服务启动的时候再将aof中的所有命令再执行一遍就可恢复数据
```
aof流程
```
1.redis接收到一个写命令
2.将该命令写入到aof_buf缓冲区
3.调用file.write()将缓冲区中的内容写入到文件中
4.依配置判断是否调用file.sync
配置文件中的appendfsync参数：
no：不进行同步，操作系统自动操作，最快
always：每次有写操作都进行同步，最慢但最安全
everysec：对写操作进行积累，每秒同步一次，默认选项
```
aof重写
1.    当服务器接收到bgrewriteaof命令时，服务器会fork一个子进程
2.    子进程遍历所有的键值对，然后生成一条可以创建该键值对的命令，然后将该命令写入到新的aof文件中
3.    而此时父进程可以继续处理请求，并将写命令写到旧的aof文件中，并且将该命令写到一个aof重写缓冲区中
4.    当子进程生成新的aof文件完成后，会通过信号通知父进程，父进程将aof缓冲区的命令写入到新的aof文件中，
并且用新的aof文件替换掉旧的aof文件
```
auto-aof-rewrite-percentage=100
auto-aof-rewrite-min-size=64mb
当执行完一次bgrewriteaof后，redis会记录此时aof文件的大小（auto_aofrewrite_base_size）
然后当每次有内容写入到aof文件时会将aof文件当前的大小保存在appendonly_current_size变量中
serverCron时间事件中每次都会检查(appendonly_current_size*100/base) - 100 值与配置中的auto-aof-rewrite-percentage值，如果大于auto-aof-rewrite-percentage则会自动重写
auto-aof-rewrite-min-size 当前文件大小的最小值，这里的”64mb”意为当前日志超过64mb时，将可能触发重建日志操作。要注意的是：auto-aof-rewrite-percentage和auto-aof-rewrite-min-size必须同时满足才可以触发重建日志操作
```

####redis转储
-    普通的copy RDB文件
-    [redis-dump](https://github.com/delano/redis-dump)

####redis集群方案
######codis   
[codis文档](https://github.com/wandoulabs/codis/blob/master/doc/tutorial_zh.md)

######twemproxy

```
能顺利解决redis单点故障不能解决自身的单点
twemproxy高可用方案需要keepalived
不能够平滑的扩容、缩容（后端增加或者删除redis单点需要重启）
```
######Cluster

######redis全量同步(sync)
主从同步流程

1. 每次slave连接到master后,master bgsave到rdb文件
2. master发送rdb文件到客户端
3. slave收到rdb文件后复制
4. master开始向slave发送写命令

slave同步过程中的状态

1. redis_repl_none
2. redis_repl_connect
3. redis_repl_connecting
4. redis_repl_receive_pong
5. redis_repl_transfer
6. redis_repl_connected

master同步过程中的状态

1. redis_repl_wait_bgsave_start
2. redis_repl_wait_bgsave_end
3. redis_repl_send_bulk
4. redis_repl_online

######redis增量同步(psync)

####参考资料
[redis资料汇总](http://blog.nosqlfan.com/html/3537.html)