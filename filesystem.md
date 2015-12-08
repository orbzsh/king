##FileSystem
####VFS虚拟文件系统
```
文件系统是内核的功能，是工作在内核空间的软件，访问一个文件必须要需要文件系统的存在才可以
Linux有多种文件系统，实现各不相同，因此Linux内核向用户空间提供了虚拟文件系统这个统一的接
口来对文件系统进行操作
```

##IO Scheduler（IO调度）
####Deadline scheduler
    
    在数据吞吐量大的数据库系统中表现更好
    deadline scheduler用deadline算法保证对于既定的IO请求以最小的延迟时间

####Anticipatory scheduler

    as曾一度是Linux 2.6 kernel的IO Scheduler，anticipatory中文意思是"预料的、预想的"，简单的说：
    有个IO发生的时候，如果又有进程请求IO操作，则将产生默认的6毫秒猜测时间， 猜测下一个进程请求
    IO是要做什么，这对于随机读取会造成比较大的延迟，对数据库应用就很操蛋，这个算法可以简单理解
    成面向低速磁盘的，因为这个"猜测"实际上的目的是为了减少磁头移动时间。
####Completely Fair Queuing
    
    默认的调度算法
    cfq（完全公平队列），对于每个进程维护一个IO队列，各个进程发来的IO请求会被cfq以轮询的方式处理
    也就是每个IO请求都是公平的
####NOOP

    noop对于IO不那么操心，对所有的IO请求都用FIFO队列形式处理，默认任务IO不会存在性能问题，这也
    使得CPU也不用那么操心。

####修改调度算法

    将elevator={deadline|noop}附在grub中的kernel命令行后面
    查看cat /sys/block/sda/queue/scheduler
    修改echo deadline > /sys/block/sda/queue/scheduler

##