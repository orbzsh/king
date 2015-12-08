####PG的一些基本命令
```
\password：设置密码
\q：退出
\h：查看SQL命令的解释，例如\h select
\l：列出所有数据库
\c [database_name] ：连接其他数据库
\d：列出当前数据库的所以表格
\du：列出所有用户
\e：打开文本编辑器
\conninfo：列出当前数据库和连接的信息
initdb -D /data/pgsql/data：初始化磁盘的数据存储区
pg_ctl -D /data/pgsql/data initdb：初始化磁盘的数据存储区
```

####PG须知
```
首先数据库的目录禁止非授权用户访问
默认的客户端认证设置允许任意本地用户连接到数据库，使用trust身份认证
```

####PG启动和关闭服务流程
关闭信号
```
SIGTERM：
智能关闭模式，服务器接收到SIGTERM后，服务器不再允许新的连接，所有的活跃的会话正常完成他们的工作，
只有在所有会话都结束任务后才关闭。
如果服务器处在在线备份模式，另外等待在线备份模式不再活跃。当备份模式活跃时，仍允许新的连接。
如果服务器正在恢复，那么恢复和流复制将在所有会话终止后停止。
SIGINT：
快速关闭模式，不再允许新的连接，向所有活跃服务发送SIGTERM，等待所有子进程退出并关闭数据库。
如果服务器处在备份模式，备份模式失败，备份无效。
SIGQUIT：
立即关闭模式，主postgres进程向所有子进程发送SIGQUIT并且立即退出，不会优雅关闭数据库
```

####PG服务所有进程
postmaster process
checkpointer process
syslogger process / logger process（系统日志进程）
```
日志进程是一个可选进程，默认关闭
```
writer process（后台写进程）
```
后台写进程是一个强制性进程
所有PG服务器进程从磁盘读取数据然后将他们移植到共享内存池（shared buffer pool）里。
共享内存池使用LRU（最近最少使用）机制来淘汰数据，BGWRITER后台写进程大多时候都在休眠，
但每次唤醒，他通过搜索共享内存缓冲池来寻找被修改的页。每次搜索完毕，BGWRITER就会选择被修改的页，
将他们写到磁盘，然后将他们从共享缓冲池里淘汰出来。
后台写进程通过BGWRITER_DELAY、BGWRITER_LRU_PERCENT、BGWRITER_LRU_MAXPAGES来控制
```
wal writer process（预写式日志写进程）
```
预写式日志写进程是一个强制性进程
预写式日志写进程在适当间隔会写入并进行文件同步，为了保证事务安全，预写式日志缓冲区在事务日志里有数据库的更改操作
预写式日志缓冲区在每次事务提交时写到磁盘，预写式日志写进程负责写到磁盘，WAL_WRITER_DELAY参数用于调用写进程
```
autovacuum launcher（自动清理启动器进程）
```
自动清理进程是一个可选进行，默认开启
为了自动执行VACUUM和ANALYZE命令，自动清理启动器进程是由多个autovacuum workers组成的后台进程
```
archiver process（归档进程）

[参考文档](http://www.enterprisedb.com/docs/en/9.0/pg/continuous-archiving.html)

```
归档进程是可选进程，默认关闭

1、在数据库归档模式，一旦预写日志（WAL）数据填满了预写日志段文件，填充满的段文件会被预写式日志

写进程在目录$PGDATA/pg_xlog/archive_status目录下创建一个后缀为.ready的文件

2、归档进程就会触发去查找那些被预写式日志进程创建的.ready状态的文件，归档进程选择这些文件，然后

从$PGDATA/pg_xlog复制这些文件到archive_command参数中指定的目录

3、成功地从源目录复制到目的目录，归档进程会重命名.ready文件为.done文件，完成归档过程

```
stats collector process（状态收集进行）
```
状态收集进程是可选进程，默认开启
状态收集进程会收集关于服务器运行的信息，他会计算访问表格索引两者磁盘快的数量和个别的行项数，同样会跟踪每一个表的
总行数，当然收集这些数据会有额外的开销。
select ctid from tbname;第一个数字就是block数，第二个是row item数
```

####PG压力测试
pgbench
I/O schedulers选择（cfq、noop、deadline）
```
-c 并发连接的客户端
-T 压测时间
-r 显示每个命令的平均延迟
pgbench -i pgbench （初始化测试数据）
```
####PG备份恢复
######逻辑备份
**pg_dump**支持4种格式（**t**|**c**格式的备份与pg_restore工具兼容、**p**格式的备份与psql工具兼容）：
pg_dump数据库级别的备份，dump过程不会拒绝连接
pg_dumpall集群级别的备份

    plain text（默认的纯文本SQL格式）
    custom（自定义格式）
    tar（打包格式）
    directory（目录格式）
纯文本SQL格式的备份和还原

    pg_dump -U pguser -Fp dbname > filename
    or
    pg_dump -U pguser dbname -f filename
    or
    pg_dump -Fp -U pguser dbname -f filename
    使用psql命令来还原
    psql -U pguser -f filename dbname
    or
    postgres=# \i sql-file-name

自定义格式

    pg_dump -Fc dbname -f filename
    pg_restore -Fc -u pguser -d dbname filename.dmp
    pg_dump -Ft dbname -f filename
    pg_restore -U pguser -d dbname filename

集群级别备份

    pg_dumpall -p por > filename
    使用psql命令来还原
    psql -f filename
######物理备份
冷备份
```
冷备份时，数据库服务需要关闭
tar zcvf backup.tar.gz $PGDATA
or
cp -r $PGDATA /backup/
or
rsync -a $PGDATA /backup/
```

在线热备份
```

postgres=# select pg_start_backup('lable')

cp -r $PGDATA /somethere

postgres=# select pg_stop_backup()

PG没有目录来保存在线热备份的开始时间和结束时间

通过pg_start_backup('lable')会创建一个文件backup_lable在$PGDATA目录下，通过pg_stop_backup()会 创建

一个文件wal-segment-number.backup在$PGDATA/pg_xlog目录下，backup_lable会给出开始时间以及wal的

检查时间点，也会通知PG实例处于备份模式，在$PGDATA/pg_xlog目录下的wal-segment-number.bakcup文件

描述了开始和停止时间，带有wal段号的检查点的位置。

在pg_stop_backup()之后，PG实例就会删除backup_lable文件

```


######PG备份脚本(归档模式)
首先修改PG配置文件
```
wal_level=archive
wal_keep_segments=1
archive_mode=on
archive_command='test -f /data/backup/inc/%f  && cp -i %p /data/backup/inc/%f'
```
####barman备份工具
barman依赖的python包
```
argh
argparse
argcomplete
python-dateutil
psycopg2
```
/etc/barman/barman.conf配置文件
```
mkdir -p /var/lib/barman && chown postgres.postgres /var/lib/barman
mkdir -p  /var/log/barman  && chown postgres.postgres /var/log/barman
touch /var/log/barman/barman.log && chown postgres.postgres /var/log/barman/barman.log

[barman]
barman_home = /var/lib/barman
barman_user = postgres
log_file = /var/log/barman/barman.log
compression = gzip
reuse_backup = link

[main]
description = "Local PostgreSQL Database"
ssh_command = ssh postgres@127.0.0.1
conninfo = host=127.0.0.1 user=postgres
minimum_redundancy = 1
retention_policy = REDUNDANCY 2

[2200420002]
description = "2200420002"
ssh_command = ssh postgres@10.4.8.40
conninfo = host=10.4.8.40 user=postgres
minimum_redundancy = 1
retention_policy = REDUNDANCY 2
```
####全备脚本示例
```
#!/bin/bash
#
PGDUMP=$(which pg_dump)
PGUSER="postgres"
PGPASS=""
PGPORT="5432"
BACK_ALL="/data/backup/all/"
BACK_INC="/data/backup/inc/"
LOG_FILE="/var/log/pgbackup.log"
INNER_IP=$(ifconfig | grep -E '([0-9]+)\.([0-9]+)\.([0-9]+)\.([0-9]+)'| awk '{print $2}' | cut -d":" -f2 | grep -E "^(192|10)\.?\.")
DATE=$(date '+%F_%H-%M')

function out_log() {
	#日志格式输出
	if [ -n "$1" ];then
		_PATH=$1
	else
		echo "Unknow Error"
		echo -e "Errro\nUnknow Error" > ${LOG_FILE}
		exit
	fi	
}

function all_back() {
	#全备，每隔4个小时全备，增备采用PG内部热增备模式
	echo "all backup"
	out_log "${PGDUMP} -U ${PGUSER} > ${BACK_ALL}/${INNER_IP}_${DATE}_${PGPORT}"
	${PGDUMP} -U ${PGUSER} > ${BACK_ALL}/${INNER_IP}_${DATE}_${PGPORT}
	cd ${BACK_ALL}
	out_log "tar zcvf ${INNER_IP}_${DATE}_${PGPORT}.tar.gz ${INNER_IP}_${DATE}_${PGPORT}"
	tar zcvf ${INNER_IP}_${DATE}_${PGPORT}.tar.gz ${INNER_IP}_${DATE}_${PGPORT}
}

```
####PG的RPM包的spec文件参考
```
%define    pgdir	/data/pgsql
%define    pgdata	/data/pgsql/data
Name:       postgresql
Version:    9.2.7
Release:	1%{?dist}
Summary:    PostgreSQL is a powerful,open source object-relational database system

Group:		Applications/Databases
License:	BSD
URL:		http://www.postgresql.org/
Source0:	http://ftp.postgresql.org/pub/source/v9.2.7/postgresql-9.2.7.tar.bz2
Source1:	postgresql
BuildRoot:	%(mktemp -ud %{_tmppath}/%{name}-%{version}-%{release}-XXXXXX)

Requires:			logrotate,perl
BuildRequires:		readline-devel,zlib-devel
Requires(post):		chkconfig
Requires(postun):	initscripts
Requires(pre):		shadow-utils
Requires(preun):	chkconfig
Requires(preun):	initscripts

%description
PostgreSQL is a powerful,open source object-relational database system

%prep
%setup -q

%build
export DESTDIR=%{buildroot}
./configure --prefix=${pgdir}
make 
mkdir -p /data/pgsql/data

%install
rm -rf %{buildroot}
make DESTDIR=%{buildroot}/%{pgdir} install
install -p -d -m 0644 %{buildroot}%{pgdir}/data
install -p -D -m 0644 %{SOURCE1} %{buildroot}%{_initrddir}/postgresql

%clean
rm -rf %{buildroot}

%pre
/bin/grep -q postgres /etc/passwd || %{_sbindir}/useradd postgres >/dev/null 2>&1

%preun
/etc/init.d/postgresql stop > /dev/null 2>&1
/sbin/chkconfig --del postgresql
/usr/sbin/userdel -f postgres > /dev/null 2>&1
/bin/rm /etc/init.d/postgresql -f >/dev/null 2>&1

%post
/bin/chown -R postgres /data/pgsql/data
su - postgres -c "/data/pgsql/bin/initdb -D /data/pgsql/data"
grep postgre /etc/profile >/dev/null 2>&1 || echo '''PATH=$PATH:/data/pgsql/bin; export PATH''' >>/etc/profile 
source /etc/profile
/bin/cp /data/pgsql/lib/libpq.so.5.5 /usr/lib64/libpq.so.5
/sbin/chkconfig --add postgresql
/bin/chmod +x /etc/init.d/postgresql

%postun
/bin/rm -rf /home/postgres > /dev/null 2>&1
/bin/rm -rf /data/pgsql/data >/dev/null 2>&1
/bin/userdel posrgres >/dev/null 2>&1
/bin/rm -f /usr/lib64/libpq.so.5 >/dev/null 2>&1
/bin/sed -i '/pgsql\/bin/d' /etc/profile
source /etc/profile

%files
%defattr(-,root,root,-)
/data/pgsql
/data/pgsql/data
%{_initrddir}/postgresql
%doc

%changelog

```

####PG重启脚本
```
#! /bin/sh

# chkconfig: 2345 98 02
# description: PostgreSQL RDBMS

# This is an example of a start/stop script for SysV-style init, such
# as is used on Linux systems.  You should edit some of the variables
# and maybe the 'echo' commands.
#
# Place this file at /etc/init.d/postgresql (or
# /etc/rc.d/init.d/postgresql) and make symlinks to
#   /etc/rc.d/rc0.d/K02postgresql
#   /etc/rc.d/rc1.d/K02postgresql
#   /etc/rc.d/rc2.d/K02postgresql
#   /etc/rc.d/rc3.d/S98postgresql
#   /etc/rc.d/rc4.d/S98postgresql
#   /etc/rc.d/rc5.d/S98postgresql
# Or, if you have chkconfig, simply:
# chkconfig --add postgresql
#
# Proper init scripts on Linux systems normally require setting lock
# and pid files under /var/run as well as reacting to network
# settings, so you should treat this with care.

# Original author:  Ryan Kirkpatrick <pgsql@rkirkpat.net>

# contrib/start-scripts/linux

## EDIT FROM HERE

# Installation prefix
prefix=/data/pgsql

# Data directory
PGDATA="/data/pgsql/data"

# Who to run the postmaster as, usually "postgres".  (NOT "root")
PGUSER=postgres

# Where to keep a log file
PGLOG="$PGDATA/serverlog"

# It's often a good idea to protect the postmaster from being killed by the
# OOM killer (which will tend to preferentially kill the postmaster because
# of the way it accounts for shared memory).  Setting the OOM_SCORE_ADJ value
# to -1000 will disable OOM kill altogether.  If you enable this, you probably
# want to compile PostgreSQL with "-DLINUX_OOM_SCORE_ADJ=0", so that
# individual backends can still be killed by the OOM killer.
#OOM_SCORE_ADJ=-1000
# Older Linux kernels may not have /proc/self/oom_score_adj, but instead
# /proc/self/oom_adj, which works similarly except the disable value is -17.
# For such a system, enable this and compile with "-DLINUX_OOM_ADJ=0".
#OOM_ADJ=-17

## STOP EDITING HERE

# The path that is to be used for the script
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# What to use to start up the postmaster.  (If you want the script to wait
# until the server has started, you could use "pg_ctl start -w" here.
# But without -w, pg_ctl adds no value.)
DAEMON="$prefix/bin/postmaster"

# What to use to shut down the postmaster
PGCTL="$prefix/bin/pg_ctl"

set -e

# Only start if we can find the postmaster.
test -x $DAEMON ||
{
	echo "$DAEMON not found"
	if [ "$1" = "stop" ]
	then exit 0
	else exit 5
	fi
}


# Parse command line parameters.
case $1 in
  start)
	echo -n "Starting PostgreSQL: "
	test x"$OOM_SCORE_ADJ" != x && echo "$OOM_SCORE_ADJ" > /proc/self/oom_score_adj
	test x"$OOM_ADJ" != x && echo "$OOM_ADJ" > /proc/self/oom_adj
	su - $PGUSER -c "$DAEMON -D '$PGDATA' &" >>$PGLOG 2>&1
	echo "ok"
	;;
  stop)
	echo -n "Stopping PostgreSQL: "
	su - $PGUSER -c "$PGCTL stop -D '$PGDATA' -s -m fast"
	echo "ok"
	;;
  restart)
	echo -n "Restarting PostgreSQL: "
	su - $PGUSER -c "$PGCTL stop -D '$PGDATA' -s -m fast -w"
	test x"$OOM_SCORE_ADJ" != x && echo "$OOM_SCORE_ADJ" > /proc/self/oom_score_adj
	test x"$OOM_ADJ" != x && echo "$OOM_ADJ" > /proc/self/oom_adj
	su - $PGUSER -c "$DAEMON -D '$PGDATA' &" >>$PGLOG 2>&1
	echo "ok"
	;;
  reload)
        echo -n "Reload PostgreSQL: "
        su - $PGUSER -c "$PGCTL reload -D '$PGDATA' -s"
        echo "ok"
        ;;
  status)
	su - $PGUSER -c "$PGCTL status -D '$PGDATA'"
	;;
  *)
	# Print help
	echo "Usage: $0 {start|stop|restart|reload|status}" 1>&2
	exit 1
	;;
esac

exit 0
```
####PG参考资料
[官方中文手册](http://www.postgres.cn/docs/9.4/index.html)
[德哥技术博客](http://blog.163.com/digoal@126/)
[谭峰技术博客](http://francs3.blog.163.com/)
[陈立群技术博客](http://my.oschina.net/Kenyon)
[唐成技术博客](http://blog.osdba.net/)
[大肚熊技术博客](http://www.cnblogs.com/daduxiong/category/257029.html)
[beigang技术博客](http://blog.csdn.net/beiigang)
[Stepgen_Liu技术博客](http://www.cnblogs.com/stephen-liu74/category/343171.html)
[David_Tang技术博客](http://www.cnblogs.com/mchina/tag/postgresql/)
[zhiyong yang技术博客](http://dreamer-yzy.github.io/tags/PostgreSQL/)
