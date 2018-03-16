---
layout: blog
istop: true
title: "centos 安装 monogdb"
background-image: 
date:  2017-08-11
category: php
tags:
- github
- mongodb
---
安装 mongodb
yum 仓库配置
vi /etc/yum.repos.d/mongodb.repo
内容为：
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/...$releasever/mongodb-org/3.2/x86_64/
gpgcheck=0
enabled=1
安装mongodb:
yum install -y mongodb-org
使用mongodb:
启动:
service mongod start
停止:
service mongod stop
重启：
service mongod restart
客户端:
mongo
配置远程连接
添加用户
执行
mongo
use admin
接着我们创建一个用户，Mongodb在3.0版本时候去除了addUser这个方法，我们需要这样来创建用户
db.createUser(
   {
     user: "<username>",
     pwd: "<password>",
     roles: [ "readWrite", "dbAdmin" ]
   }
)
//<username> 用户名
//<password> 密码
配置mongodb.conf
vim /etc/mongod.conf
修改下面配置
bindIp = 127.0.0.1 //注释此行
重启mongodb
serivce mongod restart
测试连接
打开本地命令
mongo 你的服务器ip地址:27017/admin -u <用户> -p <密码>
