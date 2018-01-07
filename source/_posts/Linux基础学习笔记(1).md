---
title: Linux基础学习笔记(1)
date: 2018-01-07 16:02:25
tags:
---

## 安装虚拟机
> 虚拟机对于初上手Linux系统的人来说，是最经济的选择。能够省去买云服务器的费用，也能从Linux的安装开始学习Linux。在工作中也能从Windows或者Mac无缝切换到虚拟机环境。

- [virtual box下载](http://download.virtualbox.org/virtualbox/5.2.4/VirtualBox-5.2.4-119785-Win.exe)
- [centos镜像](http://mirrors.163.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso)
- 其他事项
    - 虚拟机配置（用于开发）：redhat64位、1g内存、40g硬盘
    - 点击虚拟机界面后，鼠标会独占，右ctrl能取消独占
    - 设置windows与虚拟机之间的复制粘贴：设备-共享粘贴板-双向。（好像无效，还是用ssh吧。）
    - [mirrors.163.com](mirrors.163.com)是网易的镜像站，含有大量Linux镜像资源,免除国内访问国外资源很慢的烦恼。

## 准备工作
> Minimal的镜像是最小系统镜像，不包含网络等其他软件，需要自己配置。

- ifconfig（查看网络配置）
    - yum install net-tools (安装该工具使ifconfig命令可用)
- ip addr（查看ip地址）
- vi /etc/sysconfig/network-scripts/ifcfg-xx 
    - xx在ip addr的2：{网卡名}
    - 编辑ONBOOT=yes
    - 命令： service network restart
- yum需要先替换默认源：[步骤](http://mirrors.163.com/.help/centos.html)
    - 内网ip换成局域网ip。虚拟机先关机，设置-网络-连接方式改成桥接。得到局域网ip可用于虚拟机ssh连接。
    - wget（yum install wget）
    - mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
    - cd /etc/yum.repos.d/
    - wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
    - yum clean all; yum makecache
    
## SSH服务
> SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。
#### 安装
- 安装SSH yum install openssh-server
- 启动 service sshd start
- 设置开机运行 chkconfig sshd on
- linux平台安装SSH客户端 yum install openssh-clients(其实在安装openssh-server时已经顺便被安装好)
- 连接。 ssh {帐号}@{ip:端口(不填默认22)} -> 输入密码

#### SSH config
- config是为了方便批量管理多个ssh
- config存放在 ~/.ssh/config
- config配置语法
```
host "{名称1}"
    HostName {ip}
    User {帐号}
    Port {端口}
    
host "{名称2}"
    HostName {ip}
    User {帐号}
    Port {端口}
// 设置完就能以这种形式连接：ssh {名称}

```
#### ssh免密登录：
- ssh key（使用非对称方式生成公钥和私钥）。私钥（~/.ssh）,公钥（~/.ssh/authorized_keys）
- linux生成ssh key：
```
cd ~/.ssh
ssh-keygen
输入名字
输入密码（可空）
```
- 把.pub（公钥文件）内容放在~/.ssh/authorized_keys
- ssh-add ~/.ssh/imooc_rsa(linux需要)
- 
#### ssh安全端口（修改端口）
```
vim /etc/ssh/sshd_config
——————
Port {端口号1}
Port {端口号2}
——————
service sshd restart
```

## Linux常用命令
#### 软件操作命令
- 软件包管理器：yum
- 安装软件：yum install xxx
- 卸载软件：yum remove xxx
- 搜索软件：yum search xxx
- 清理缓存：yum clean packages
- 列出已安装：yum list
- 软件包信息：yum info xxx

#### 硬件资源信息
- 内存：free -m
- 硬盘：df -h
- 负载：w或top
    - load average： 0.00（最近1min） 0.01（最近5min） 0.05（最近10min）
    - 等于1是满负荷
    - 大于1超负荷
    - 0.6~0.7是健康值
- cpu个数和核数
    - cat /proc/cpuinfo
    - cat是查看文件内容

#### 文件操作命令
- Linux文件目录结构
    - 根目录: /
    - 家目录：/home
    - 临时目录: /tmp
    - 配置目录: /etc
    - 用户程序目录: /usr
- 文件基本操作命令
    -  查看目录下的文件: ls（ls -al == ll 显示文件详细信息）
    -  新建文件： touch
    -  新建文件夹：mkdir（-p 表示循环生成）
    -  进入目录： cd
    -  删除文件和目录：rm（-r 表示循环删除， -rf 表示强制删除）
    -  复制：cp
    -  移动： mv
    -  显示路径：pwd
- Vim
    - [教程](http://www.runoob.com/linux/linux-vim.html)
    - i键进入insert模式
    - esc退出insert模式，然后:wq(保存退出)，:q(退出)，:w(保存)
    - G（最后一行），gg（行首）
    - dd(删除本行), u(恢复)
    - yy（复制光标所在行），p（粘贴）
- 文件权限421
    - r=4(读) w=2(写) x=1(可执行)
    - drwxr-xr-x d表示文件类型，755
- 文件搜索，查找，读取
    - 从文件尾部读取：tail(-f)
    - 从文件头部读取：head
    - 读取整个文件：cat
    - 分页读取：more(enter键往下继续读)
    - 可控分页：less
    - 搜索关键字：grep(-n显示行数 "{关键字}" {文件名})
    - 查找文件：find（如：find . -name "{文件名关键字}"，-type表示查找类型，-ctime表示更新过的时间、天）
    - 统计个数：wc
    - 管道：|（把上一次操作的结果传递给下一次操作，如：grep "{关键字}" {文件名}|wc -l，统计某文件里出现某关键字的行数）
- 文件压缩解压
    - man {命令}（查看一个命令的说明）
    - 压缩、解压：tar
    ```
    -cf {文件名.tar} {文件名},创建一个压缩文件
    -tf,查看压缩文件内容
    -tvf,查看压缩文件详细内容
    -xf,解压文件
    -czvf {文件名.tar.gz} {文件名},创建一个.gz压缩文件并查看
    -tzvf， 查看一个.gz压缩文件
    -xzvf,解压一个.gz压缩文件
    ```
    
#### 系统用户命令
- 添加用户：useradd
    ```
    cd /home
    useradd {用户名}
    passwd {用户名}
    ```
- adduser（与useradd有何不同）
- 删除用户：userdel（-r，彻底删除）
- 设置密码：passwd
    
#### 防火墙设置
- 安装：yum install firewalld
- 启动：service firewalld start
- 重启：service firewalld restart
- 检查状态：service firewalld status
- 关闭或禁用防火墙：service firewalld stop/disable
- firewall-cmd
    - 移除服务：--remove-service=ssh
    - 添加服务：--add-service=ssh
    - 查询具体的某个服务：--query-service=ssh
    - 列出服务：--list-services
    - 查询端口：--query-port=22/tcp
    - 添加端口：--add-port=22/tcp
    - 列出端口：--list-ports
    - 列出区域配置情况：--list-all-zones
    - 
#### 提权和文件上传下载操作
- 提权
    - sudo
    ```    
    visudo
    增加行:%{需要提权的帐号} ALL=(ALL) ALL, 保存退出
    ```
- 文件下载
    - wget {链接}
    - curl
    ```
    curl -o {文件名} {链接}
    ```
- 文件上传
    - 从本地上传到远端：scp {文件名} {用户名}@{ip}:{路径}
    - 从远端下载到本地：scp {用户名}@{ip}:{路径/文件} {本地路径}
    - xshell上传文件
    ```
    //远端需要安装一个软件
    yum install lrzsz
    rz //弹出弹窗，选择文件就能实现上传
    sz {文件名} //下载
    ```
