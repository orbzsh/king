##本地yum源
####创建yum仓库
```
yum install -y createrepo
mkdir /data/repos
createrepo -d /data/repos
```
####定义repo文件
```
[local_repo]
name=local repo
baseurl=http://ipaddress/$releasever/$basearch/
enable=1
gpgcheck=0
```
####通过nginx代理yum网络安装方式
```
nginx_repo.conf
server {
    listen 8080;
    root /data/repos;
    access_log logs/nginx_repo.log main;
}
```
##rpm包封装

##fpm封装rpm包