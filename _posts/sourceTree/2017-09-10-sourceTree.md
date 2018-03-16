---
layout: blog
istop: true
title: "sourceTree 安装"
background-image:
date:  2018-03-11
category: sourceTree
tags:
- github
---

注意都是ssh链接


解决方案：
*首先，sourcetree无法使用设置密码账号的方式实现登录请求。也无法像windows环境下的sourcetree一样配置key文件的地址。所以我们实现的方式是在mac上配置一个秘钥，而sourcetree登录时候正式使用这个秘钥文件。
 
1、打开控制台：ssh-keygen -t rsa -C "GIT上的账号邮箱"
2、回车
3、输入密码（git上的账号密码）
4、确认密码
5、输入命令 cd .ssh
6、输入命令 cat id_rsa.pub
7、复制出现的代码，打开GIT平台，找到SSH KEY管理菜单，在对应输入框里输入刚复制的代码，保存。
 
OK，截止到这里，再重新打开SOURCETREE链接远程仓库，这时候就可以下载下来代码了。
