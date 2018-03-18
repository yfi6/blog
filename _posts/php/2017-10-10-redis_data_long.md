---
layout: blog
istop: true
title: "redis 配置数据持久化"
background-image: http://121.43.158.69/img/3.jpg
date:  2017-10-10
category: redis
tags:
- github
- redis
- php
---
配置redis.conf

dir 是redis aof文件所在的目录
　1、appendfsync no
　　当设置appendfsync为no的时候，Redis不会主动调用fsync去将AOF日志内容同步到磁盘，所以这一切就完全依赖于操作系统的调试了。对大多数Linux操作系统，是每30秒进行一次fsync，将缓冲区中的数据写到磁盘上。
　　2、appendfsync everysec
当设置appendfsync为everysec的时候，Redis会默认每隔一秒进行一次fsync调用，将缓冲区中的数据写到磁盘。但是当这一 次的fsync调用时长超过1秒时。Redis会采取延迟fsync的策略，再等一秒钟。也就是在两秒后再进行fsync，这一次的fsync就不管会执行多长时间都会进行。这时候由于在fsync时文件描述符会被阻塞，所以当前的写操作就会阻塞。
所以，结论就是：在绝大多数情况下，Redis会每隔一秒进行一次fsync。在最坏的情况下，两秒钟会进行一次fsync操作。
这一操作在大多数数据库系统中被称为group commit，就是组合多次写操作的数据，一次性将日志写到磁盘。
　　3、appednfsync always
当设置appendfsync为always时，每一次写操作都会调用一次fsync，这时数据是最安全的，当然，由于每次都会执行fsync，所以其性能也会受到影响
 　　建议采用 appendfsync everysec（缺省方式）
　　快照模式可以和AOF模式同时开启，互补影响



appendonly no或者yes
含义：Redis Server是否开启AOF持久化机制


appendfsync

含义：Redis将OS数据缓冲区中数据刷新到磁盘的策略
# appendfsync always  只要有新添加的数据就fsync
appendfsync everysec  支持延迟fsync
# appendfsync no      不需要fsync

redis 宕机后 重启 会去加载dump.rdb文件 这个文件相当于数据快照 如果有aof数据持久化的文件 会先加载aof这个文件 每次redis操作 都会进行快照
