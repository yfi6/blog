---
layout: blog
istop: true
title: "swoole 安装"
background-image: 
date:  2017-11-10
category: swoole
tags:
- swoole
---

swoole 实例化函数 not found 可能是跟系统版本有关 请注意

如果 pecl 不存在
解决方案：
yum install -y PHP-devel php-pear httpd-devel

pecl install swoole


注意： swoole 的一些类不能使用 不存在 跟swoole版本有关

列入：$http = new swoole_http_server("127.0.0.1", 9501); PHP Fatal error:  Class 'swoole_http_server' not found

协程 环境要求 swoole2.0 php 5.5版本以上
