---
title: mongodb学习笔记
date: 2018-01-07 18:31:29
tags:
---
## 安装
### 下载
```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.0.tgz
```
### 解压安装配置
```
/**  解压  **/
tar zxvf mongodb-linux-x86_64-3.4.0.tgz

/** 重命名 **/
mv mongodb-linux-x86_64-3.4.0.tgz mongodb

/** 进入mongodb目录 **/
cd mongodb

/** 创建db和日志目录 **/
mkdir /opt/mongodb/data 
mkdir /opt/mongodb/data/db
mkdir /opt/mongodb/data/logs
/** logs目录下创建mongodb.log文件 **/
touch mongodb.log

/** 在mongodb/data目中创建mongodb.conf **/
cd data
vi mongodb.conf

/** 加入相关配置 **/
#端口号
port = 27017 
#数据目录
dbpath = /opt/mongodb/data/db
#日志目录
logpath = /opt/mongodb/data/logs/mongodb.log
#设置后台运行
fork = true
#日志输出方式
logappend = true
#开启认证
#auth = true

/** esc :wq 保存退出 **/
```
### 运行
```
/opt/mongodb/bin/mongod --config /opt/mongodb/data/mongodb.conf
```
### 用户权限
- 教程： http://blog.csdn.net/dbabruce/article/details/50963956
- 创建用户
```
use admin
db.createUser({
    user:'{用户名}',
    pwd:'{用户密码}',
    roles:[{ "role" : "{角色}", "db" : "{库名}" }]
});
```
- 内置角色
```
数据库用户类：
read 非系统集合有查询权限
readWrite 非系统集合有查询和修改权限

数据库管理类：
dbAdmin 数据库管理相关，比如索引管理，schema管理，统计收集等，不包括用户和角色管理
dbOwner 提供数据库管理，读写权限，用户和角色管理相关功能
userAdmin 提供数据库用户和角色管理相关功能

集群管理类：
clusterAdmin 提供最大集群管理权限
clusterManager 提供集群管理和监控权限
clusterMonitor 提供对监控工具只读权限
hostManager 提供监控和管理severs权限

备份和恢复类：
backup 提供数据库备份权限
restore 提供数据恢复权限

All-Database类：
readAnyDatabase 提供读取所有数据库的权限除了local和config数据库之外
readWriteAnyDatabase 和readAnyDatabase一样，除了增加了写权限
userAdminAnyDatabase 管理用户所有数据库权限，单个数据库权限和userAdmin角色一样
dbAdminAnyDatabase 提供所有用户管理权限，除了local,config

超级用户类：
root 数据库所有权限

内部角色：
__system 提供数据库所有对象任何操作的权限，不能分配给用户，非常危险
```

### 命令
- 教程： https://www.cnblogs.com/PheonixHkbxoic/p/5665499.html
- 关闭
```
// 开启mongo命令行
./bin/mongo
// 使用admin权限
use admin
// 关闭服务,不kill进程
db.shutdownServer()

// 关闭进程
mongod --shutdown --dbpath /opt/mongodb/data/db
```
-  db的帮助
```
db.changeUserPassword(username,password);  修改用户密码 
db.auth(usrename,password)     设置数据库连接验证  
db.cloneDataBase(fromhost)     从目标服务器克隆一个数据库  
db.commandHelp(name)           returns the help for the command  
db.copyDatabase(fromdb,todb,fromhost)  复制数据库fromdb---源数据库名称，todb---目标数据库名称，fromhost---源数据库服务器地址  
db.createCollection(name,{size:3333,capped:333,max:88888})  创建一个数据集，相当于一个表  
db.currentOp()                 取消当前库的当前操作  
db.dropDataBase()              删除当前数据库  
db.eval(func,args)             run code server-side  
db.getCollection(cname)        取得一个数据集合，同用法：db['cname'] or  
db.getCollenctionNames()       取得所有数据集合的名称列表  
db.getLastError()              返回最后一个错误的提示消息  
db.getLastErrorObj()           返回最后一个错误的对象  
db.getMongo()                  取得当前服务器的连接对象get the server  
db.getMondo().setSlaveOk()     allow this connection to read from then nonmaster membr of a replica pair  
db.getName()                   返回当操作数据库的名称  
db.getPrevError()              返回上一个错误对象  
db.getProfilingLevel()         获取profile level  
db.getReplicationInfo()        获得重复的数据  
db.getSisterDB(name)           get the db at the same server as this onew  
db.killOp()                    停止（杀死）在当前库的当前操作  
db.printCollectionStats()      返回当前库的数据集状态  
db.printReplicationInfo()        打印主数据库的复制状态信息  
db.printSlaveReplicationInfo()        打印从数据库的复制状态信息  
db.printShardingStatus()       返回当前数据库是否为共享数据库  
db.removeUser(username)        删除用户  
db.repairDatabase()            修复当前数据库  
db.resetError()  
db.runCommand(cmdObj)          run a database command. if cmdObj is a string, turns it into {cmdObj:1}  
db.setProfilingLevel(level)    设置profile level 0=off,1=slow,2=all  
db.shutdownServer()            关闭当前服务程序  
db.version()                   返回当前程序的版本信息  
```
- 表的帮助，格式，db.表名.help()
```
db.test.find({id:10})          返回test数据集ID=10的数据集  
db.test.find({id:10}).count()  返回test数据集ID=10的数据总数  
db.test.find({id:10}).limit(2) 返回test数据集ID=10的数据集从第二条开始的数据集  
db.test.find({id:10}).skip(8)  返回test数据集ID=10的数据集从0到第八条的数据集  
db.test.find({id:10}).limit(2).skip(8)  返回test数据集ID=1=的数据集从第二条到第八条的数据  
db.test.find({id:10}).sort()   返回test数据集ID=10的排序数据集  
db.test.findOne([query])       返回符合条件的一条数据  
db.test.getDB()                返回此数据集所属的数据库名称  
db.test.getIndexes()           返回些数据集的索引信息  
db.test.group({key:...,initial:...,reduce:...[,cond:...]})    返回分组信息  
db.test.mapReduce(mayFunction,reduceFunction,<optional params>)  这个有点像存储过程  
db.test.remove(query)                      在数据集中删除一条数据  
db.test.renameCollection(newName)          重命名些数据集名称  
db.test.save(obj)                          往数据集中插入一条数据  
db.test.stats()                            返回此数据集的状态  
db.test.storageSize()                      返回此数据集的存储大小  
db.test.totalIndexSize()                   返回此数据集的索引文件大小  
db.test.totalSize()                        返回些数据集的总大小  
db.test.update(query,object[,upsert_bool]) 在此数据集中更新一条数据  
db.test.validate()                         验证此数据集  
db.test.getShardVersion()                  返回数据集共享版本号  
```
### 连接
```
mongodb://{user}:{password}@{ip}:{port}/{dbname}
```

### 数据库导出导入
- 导出：mongoexport
```
mongoexport -d dbname -c collectionname -o file --type json/csv -f field
    参数说明：
        -d ：数据库名
        -c ：collection名
        -o ：输出的文件名
        --type ： 输出的格式，默认为json
        -f ：输出的字段，如果-type为csv，则需要加上-f "字段名"
```
- 导入：mongoimport
```
mongoimport -d dbname -c collectionname --file filename --headerline --type json/csv -f field
    参数说明：
        -h [ --host ] arg       mongo host to connect to ( <set name>/s1,s2 for sets)  
        --port arg              server port. Can also use --host hostname:port   
        -u [ --username ] arg   username  
        -p [ --password ] arg   password  
        -d ：数据库名
        -c ：collection名
        --type ：导入的格式默认json
        -f ：导入的字段名
        --headerline ：如果导入的格式是csv，则可以使用第一行的标题作为导入的字段
        --file ：要导入的文件
```
- tips，下载远程文件
```
scp {用户名}@{ip}:{路径/文件} {本地路径}
```