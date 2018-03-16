---
layout: blog
istop: true
title: "配置多喝虚拟机 报错"
background-image: 
date:  2017-11-12
category: nginx
tags:
- github
- nginx
---
一开始只起了一个php-cgi进程，结果发现进入主界面后，每次点“配置”后，系统就会阻塞，直到超时后报错“PDOException: SQLSTATE[23000]: Integrity constraint violation: 1048 Column &#039;uid&#039; cannot be null", 这里应该是$uid取值为空。
首先到db里看了user表和session表，发现有内容。没办法又根据关键字找了一圈代码，也没有发现有用的线索。后来在群里经过好心人的提醒，cgi处理进程要启多个，因为“有些程序会curl,访问自己,系统就会卡死,或通信失败”。然后又启了1个php-cgi（现在有2个），问题解决。
这里附上nginx.conf里配置多个fastcgi的指令吧，希望能给遇到类似问题的同学一点帮助：
nginx.conf
>> 
?
13http
{
 upstream myfastcgi {
 server 127.0.0.1:9000 weight=1;
 server 127.0.0.1:9001 weight=1;
 }
 
 server {
 localtion ~ \.php$ {
 fastcgi_pass myfastcgi;
 }
 }
}
