---
layout: blog
istop: true
title: "swoole redis"
background-image: 
date:  2017-11-11
category: swoole
tags:
- github
- swoole-redis
---
https://wiki.swoole.com/wiki/page/325.html
$serv = new swoole_server("0.0.0.0", 9502);

//必须在onWorkerStart回调中创建redis/mysql连接
$serv->on('workerstart', function($serv, $id) {
    $redis = new redis;
    $redis->connect('127.0.0.1', 6379);
    $serv->redis = $redis;
});

$serv->on('receive', function (swoole_server $serv, $fd, $from_id, $data) { 
    $value = $serv->redis->get("key");
    $serv->send($fd, "Swoole: ".$value);
});

$serv->start();
