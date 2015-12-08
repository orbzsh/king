####agent模式
主动模式（agent向server提交数据）

[active check system.run is not supported](https://support.zabbix.com/browse/ZBX-1256)
[system.run does not work in Linux](https://www.zabbix.com/forum/showthread.php?t=12352)

```
在指定的主机上运行命令
system.run[command,<mode>]
command    命令
mode    wait（默认值，执行命令超时时间），nowait（不等待）
```

被动模式（server向agent讨要数据）
####新加项目流程
1.    新建主机群组
2.    新建模板并关联群组
3.    链接基础模板（可以不用链接，单独定制）
4.    定制模板（创建应用集、创建监控项、配置触发器）
5.    创建动作
6.    自动注册

####trigger注意
```
count(#3,1,"eq") = 1
表示最近3次的值等于1，触发1次就会报警
```
####zabbix-postgresql监控指标
-    服务是否运行
-    连接数
-    慢查询

模板巨集定义
```
{$PGHOST}=127.0.0.1
{$PGUSER}=postgres
{$PGDATABASE}=postgres
{$PGPORT}=5432
{$PGSCRIPTDIR}=/usr/local/bin
```

项目定义
```
psql.running[${PGHOST},${PGUSER},${PGPORT},${PGDATABASE}]
```

zabbix-agent.conf
```
UserParameter=psql.running[*],/data/pgsql/bin/psql -h $1 -U $2 -p $3 -d $4 -c "select 1">/dev/null 2>&1;echo $?
```
####zabbix-redis监控指标
####zabbix-mysql监控指标
####zabbix-nginx监控指标
####zabbix-php监控指标