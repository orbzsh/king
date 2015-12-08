####tunnel的实现
```
隧道的实现主要基于两个文件new_tunnel.c和ipip.c
同时定义一种新的协议类型：IPIP
```
####linux与linux之间的tunnel例子
######linux_server_1(111.111.111.111、192.168.10.0/24)
```
modprobe ipip
ip tunnel add tunl mode ipip remote 222.222.222.222 local 111.111.111.111 ttl 64
ip link set tunl mut 1480 up
ip address add 192.168.10.253 brd 255.255.255.255 peer 222.222.222.222 dev tunl
ip route 192.168.10.0/24 via 192.168.10.253
```
######linux_server_2(222.222.222.222、192.168.20.0/24)
```
modprobe ipip
ip tunnel add tunl mode ipip remote 111.111.111.111 local 222.222.222.222 ttl 64
ip link set tunl mut 1480 up
ip address add 192.168.20.253 brd 255.255.255.255 peer 111.111.111.111 dev tunl
ip route 192.168.20.0/24 via 192.168.20.253
```
######参数说明
1.    mtu隧道会增加协议开销，因为它需要一个额外的IP包头
2.    ttl数据包的生存周期