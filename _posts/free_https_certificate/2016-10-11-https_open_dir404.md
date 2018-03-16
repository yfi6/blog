---
layout: blog
istop: true
title: "服务器opendir_dir 404"
background-image: 
date:  2016-10-11
category: https
tags:
- 服务器问题
---
今天偶然发现，在php脚本中可以访问服务器上任何目录。
顿时一身冷汗啊。。
于是就上百度谷歌了半天，一开始就找到了一个php.ini中的open_basedir参数，设置这个参数即可限定php脚本的访问范围。
我们针对每个站点，需要php能够访问该站点所在目录以及/tmp/临时目录。
SO..看到有人这么写
open_basedir=.:/tmp/  
冒号的作用是隔开多个路径，这里面根据字面理解，第一个点就代表当前目录。
看起来是很完美了，OK，保存配置，重启php-fpm
结果nginx 报502错误。
研究了一会，发现 . 这种相对路径写法，至少在nginx+phpfastcgi下是行不通的。
好吧，暂时妥协
open_basedir=/home/wwwroot/:/tmp/
这样总行了，将php限制在所有站点的父目录，这样至少阻止了php访问服务器上web目录以外的目录。
到了这里，还是有隐患的，只要wwwroot下任意一个站点被拿到webshell，那么其他站点将不能幸免.
不甘心哪，度娘是找不到有用的信息了，都是些垃圾复制粘贴，于是去了谷歌。
搜了一下，找到一个遇到同样问题的鬼佬，里面有人给了一个方法，成功解决。
那就是在nginx 每个server下，加上
fastcgi_param  PHP_VALUE  "open_basedir=$document_root:/tmp/"; 
重启nginx，成功！你也可以把这行代码放到fastcgi.conf里，前提是你得在server{}中包含它。
至此，nginx + php5.3 是没有问题了。
然后我又在另外一台vps上，环境是php5.2 发现此方法不生效。
