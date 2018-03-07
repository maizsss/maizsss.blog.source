---
title: 在linux环境部署nodejs项目
date: 2018-03-07 20:44:34
tags:
---


> 背景：当你做了一个nodejs+mongodb的项目，然后拿到了linux云主机的ip与root密码，接着要如何完成一系列的部署工作。

## linux用户权限
- 因为root账户具有系统最高权限，如果进行了误操作或者泄露了，后果会很严重。所以一般会根据不同的操作内容建立多个非root账户，分配不同的权限。
- linux的权限系统还有文件权限、用户组、提权这样的概念，理解了linux的权限系统可以更自如地进行操作和定位问题，具体可以参考文章来了解： https://www.cnblogs.com/duhuo/p/5892513.html

## 系统构成
- 简单描绘一个基本的系统架构
![image](http://7xpsli.com1.z0.glb.clouddn.com/linux_deploy_pic1.png)

# 软件环境准备

## 安装nodejs
- 使用nvm管理nodejs版本。nvm方便在同一台机器安装不同版本nodejs，而且可以在不同版本之间进行切换。
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash
source ~/.bashrc
```
- 安装node 8.9.0
```
nvm install v8.9.0
```
## 安装git
> 使用编译安装方式，可以安装较新版本的git
- 安装依赖的包
```
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```
- 下载git源码并解压
```
wget https://github.com/git/git/archive/v2.3.0.zip
unzip v2.3.0.zip
cd git-2.3.0
```
- 编译安装
```
make prefix=/usr/local/git all
make prefix=/usr/local/git install
```
- 配置环境变量
```
// 查看git路径
whereis git
vim ~/.bashrc
// 在文件的最后一行，添加下面的内容，然后保存退出。
export PATH=/usr/local/git/bin:$PATH
// 使配置即时生效
source ~/.bashrc
```
## 设置sshkey
```
// 在linux服务端生成sshkey
ssh-keygen -t rsa -C "你的账号"
// 生成过程按回车，以默认值即可
// 最终生成在~/.ssh/id_rsa(私钥)  ~/.ssh/id_rsa.pub(公钥)
// 复制公钥的值
cat ~/.ssh/id_rsa.pub
// 在gitlab部署密钥处，添加新的公钥。
// 以ssh方式拉取项目：git clone git@XXXXX
```
## 安装mongodb
- https://maizsss.github.io/2018/01/07/mongodb%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/
## Nginx
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

## PM2：nodejs的守护进程工具
- 安装,pm2是npm包，可以直接用npm全局安装。
```
npm i pm2 -g
```
- 启动项目
```
pm2 start app.js
// 查看参数说明
pm2 help
```
- 以配置文件方式启动
```
pm2 start ecosystem.config.js --env production
```

# 项目部署
> linux环境安装软件的工作一般是运维人员处理好，而针对单个项目的部署才是开发者需要做的。
## 在git项目开启ssh秘钥
- 以gitlab为例，如果之前已经添加过服务器的密钥，那只需要在项目的“设置”下拉菜单中，进入“部署密钥”，然后在里面找到需要部署项目的服务器的密钥，点击“启用”就能以ssh方式拉取项目。
## 运行nodejs实例
- 主流做法是用pm2启动：参考上一节pm2的说明。
- 用nohub启动:nohup是linux环境的软件，能让进程在后台运行。
```
nohup node app.js &
```
- 用forever启动：具体参考（https://github.com/foreverjs/forever）
## 做nginx配置
- nginx的说明参考上一节，或：http://www.nginx.cn/doc/
- nginx本身是一个性能优秀的web服务器，用于代理静态资源是很方便的。对于nodejs服务，能解决前后端分离的跨域问题，也能实现负载均衡。
## 防火墙通行端口
- 假如nginx配置了虚拟主机占用8000端口，然后在本机curl http://localhost:8000是有响应的，但在其他机器上面访问却无响应。这时候可能是因为服务器开了防火墙而8000端口没被放行。
- 添加端口
```
// 永久添加端口
firewall-cmd --permanent --zone=public --add-port=8000/tcp
// reload配置
firewall-cmd --reload
```
