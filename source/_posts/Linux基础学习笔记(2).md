---
title: Linux基础学习笔记(2)
date: 2018-01-07 18:30:15
tags:
---
## WebServer
#### Apache
##### 基本
- 安装： yum install httpd
- 启动：service httpd start
- 停止：service httpd stop
- 看进程：ps -ef | grep httpd
- 网络统计：sudu netstat -anpl | grep 'http'
- 
##### 虚拟主机配置

```
// 编辑配置需要先提权
vim /etc/httpd/conf/httpd.conf
// 编写内容
<VirtualHost *:80>
    ServerName {域名}
    DocumentRoot {根路径}
    <Directory "{根路径}">
        Options Indexed FollowSymLinks
        AllowOverride none
        Require all granted
    </Directory>
</VirtualHost>
// 保存重启
```
- 把某个文件为帐号提权：sudo chown -R {帐号}:{帐号} {目录}
- 开启宽松模式：sudo setenforce 0 or vim /etc/selinux/config

##### 伪静态
- 模块配置文件目录：/etc/httpd/conf.modules.d/
- 模块目录：/etc/httpd/modules/
- 配置伪静态：
```
vim /etc/httpd/conf/httpd.conf

// 把伪静态加载
LoadModule rewrite_module modules/mod_rewrite.so
// 修改虚拟主机内容
<VirtualHost *:80>
    ServerName {域名}
    DocumentRoot {根路径}
    <Directory "{根路径}">
        Options Indexed FollowSymLinks
        AllowOverride none
        Require all granted
        // 加入以下内容
        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteRule ^(.*).htmp$ index.html
        </IfModule>
    </Directory>
</VirtualHost>
// 保存重启
```

#### Nginx
##### 安装
```
// 搜索，yum源没有nginx源
yum search nginx
// 添加CentOS 7 Nginx yum资源库
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
sudo yum install -y nginx
```
##### 基本操作
- 安装：yum install nginx
- 启动：service nginx start
- 停止：service nginx stop
- 重载：service nginx reload(无缝重载)
- 先杀死再启动：service nginx restart
##### 虚拟主机
- 配置：
```
vim /etc/nginx/conf.d/{配置名}.conf

server {
    listen 80;
    listen 9999;
    server_name {域名1} {域名2};
    root {根路径};
    index index.html index.htm;
}
```
- 伪静态
```
server {
    listen 80;
    listen 9999;
    server_name {域名1} {域名2};
    root {根路径};
    index index.html index.htm;
    
    location / {
        rewrite ^(.*)\.htmp$ /index.html;
    }
}
```
- 日志（搜索nignx log_format学习日志配置）
```
vim /etc/nginx/nginx.conf

log_format {日志格式名} {格式}
access_log {日志路径} {日志格式名};
```
- 反向代理（用户请求到A机器的nginx，nginx再请求B、C机器服务，再返回给用户）
- 负载均衡（N台机器同时接受请求）
```
upstream {服务名称} {
    server {ip}:{port} weight=5; // 服务1
    server {ip}:{port} weight=1; //服务2
    // weight设置权重
}

server {
    listen 80;
    listen 9999;
    server_name {域名1} {域名2};
    root {根路径};
    index index.html index.htm;
    
    location / {
        proxy_set_header Host {域名};
        proxy_pass http://{服务名};
    }
}
```
- 调试
```
upstream {服务名称} {
    server {ip}:{port} weight=5; // 服务1
    server {ip}:{port} weight=1; //服务2
    // weight设置权重
}

server {
    listen 80;
    listen 9999;
    add_header Content-Type "text/plain chatset=utf-8"; //设置header
    return 200 "$http_host"; // 直接返回给浏览器
    server_name {域名1} {域名2};
    root {根路径};
    index index.html index.htm;
    
    location / {
        proxy_set_header Host {域名};
        proxy_pass http://{服务名};
    }
}
```

## 缓存服务

### memcached
- 安装：yum install memcached
- 启动：memcached -d -l -m -p
```
// 查看网络服务
netstat -anpl | grep memcached
// 查看端口是否通,还能发送报文
// 先安装
yum install telnet.*
telnet 127.0.0.1 11211
// quit 退出
```
- 停止：kill {pid}

### redis
- 安装：源码编译安装
```
// 下载
wget http://download.redis.io/releases/redis-4.0.2.tar.gz
// 解压
tar xvfz redis-4.0.2.tar.gz
cd redis-4.0.2
// 查看安装说明
cat INSTALL
cat README.md
// 编译
// 需要先安装gcc
yum install gcc
make MALLOC=libc
// 安装
make install
```
- 启动：./src/redis-server start/restart
- 停止：./src/redis-server top
- 客户端：./src/redis-client(cli)
```
set {key} {value}
get {key}
del {key}
```
```
telnet {host} 6379
// 需要先 
CONFIG SET protected-mode no
```
- LPUSH
```
//push
LPUSH tmp_list item1
LPUSH tmp_list item2
LPUSH tmp_list item3
//range
LRANGE tmp_list 0 10
```
- redis 菜鸟教程：http://www.runoob.com/redis/redis-commands.html

## 常用服务
### crontab 定时任务
- 配置任务：crontab -e
- 教程： https://www.cnblogs.com/intval/p/5763929.html

### Ntpdate 日期同步
- 安装： yum install ntpdate
- 同步时间：ntpdate cn.pool.ntp.org
- 改时区
```
//删除原时区
rm /etc/localtime
// 设置东八区时区
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### Logrotate 日志切割
- 教程： https://linux.cn/article-4126-1.html

### supervisor 进程管理
- 教程：http://blog.csdn.net/xyang81/article/details/51555473

## shell
- 简介
> Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。
> Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。
> Ken Thompson 的 sh 是第一种 Unix Shell，Windows Explorer 是一个典型的图形界面 Shell。

- Shell 脚本
> Shell 脚本（shell script），是一种为 shell 编写的脚本程序。
> 业界所说的 shell 通常都是指 shell 脚本，但读者朋友要知道，shell 和 shell script 是两个不同的概念。
> 由于习惯的原因，简洁起见，本文出现的 "shell编程" 都是指 shell 脚本编程，不是指开发 shell 自身。

- 查看当前发行版可以使用的shell
```
cat /etc/shells 
// 输出
/bin/sh
/bin/bash
/sbin/nologin
```
- 查看当前使用的shell
```
echo $SHELL
// 输出
/bin/bash
```
- 教程：http://www.runoob.com/linux/linux-shell.html

## 别名
- 查看别名: alias
- 设置别名：alias {别名}='{原命令}'
- 永久生效：vi ~/.bashrc(写入环境变量配置文件)
```
// 使配置生效
source .bashrc
```
- 删除别名：unalias {别名}

## 常用快捷键
- 退出：ctrl+c
- 清屏：ctrl+l
- 光标移动到命令行首：ctrl+a
- 光标移动到行尾：ctrl+e
- 从光标所在位置删除到行首：ctrl+u
- 把命令放入后台：ctrl+z
- 在历史命令中搜索：ctrl+r