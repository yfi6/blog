---
layout: blog
istop: true
title: "redis 独占锁"
background-image: http://121.43.158.69/img/1.jpg
date:  2017-10-10
category: php
tags:
- php
- redis
---
1.并发问题
并发大家都知道是什么情况，这里说的是并发多个请求抢占同一个资源,直接上实例吧
请求：index.php?mod=a&action=b&taskid=6处理：
$key = "a_b::".$uid.'_'.$taskid;
$v = $redis->get($key);
if($v == 1){
    $redis->setex($key,10,1);
    //处理逻辑省略
}
2.分析
逻辑看来还可以，结果发现数据库中写入了两个同样的请求结果，我看了记录的时间戳,天!居然是同一秒. 我用microtime(true) log一下两个请求的时间差居然相差了0.0001s，就是说$redis->setex($key,10,1);还没执行成功 第二个请求已经get到跟第一个请求一样的结果。这不就是传说中的并发抢占资源。这中情况 听过很多，在开发过程中也没刻意去模拟实验过。
3.解决
方案1：第一反应就是要给处理过程加事务(数据库是mysql innoDB)，加事务的结果就是 第一个请求成功了 第二个请求会执行到后面捡查发现重了会回滚。其实mysql事务在保证数据一致性上是很ok的，但是通过回滚来保证唯一资源独占代价太大，做过mysql事务测试测同学都知道，事务中的insert是已经插进去了，回滚之后才删掉的。
方案2：还有一个选择就是php中的文件独占锁，那就是说这情况下我要新建 用户数 * 任务数的文件来实现每个请求资源的独占，如果独占资源较少的话可选的解决办法：
    /**
     * 加锁
     */
    public function file_lock($filename){
        $fp_key = sha1($filename);
        $this->fps[$fp_key] = fopen($filename, 'w+');
        if($this->fps[$fp_key]){
            return flock($this->fps[$fp_key], LOCK_EX|LOCK_NB);
        }
        return false;
    }
    /**
     * 解锁
     */
    public function file_unlock($filename){
        $fp_key = sha1($filename);
        if($this->fps[$fp_key] ){
            flock($this->fps[$fp_key] , LOCK_UN);
            fclose($this->fps[$fp_key] );
        }
    }
方案3：发现$redis->setnx()可以提供原子操作的状态：相同的key执行setnx之后没过期或者没del，再执行会返回false。这就让两个以上的并发请求得到控制必须成功获取锁才能继续。


 /**
     *  加锁
     */
    public function task_lock($taskid){
            $expire = 2;
             $lock_key ='task_get_reward_'.$this->uid.'_'.$taskid;
            $lock = $this->redis->setNX($lock_key , time());//设当前时间
            if($lock){
                $this->redis->expire($lock_key,  $expire); //如果没执行完 2s锁失效
            }
            if(!$lock){//如果获取锁失败 检查时间
                $time = $this->redis->get($lock_key);
                if(time() - $time  >=  $expire){//添加时间戳判断为了避免expire执行失败导致死锁 当然可以用redis自带的事务来保证
                    $this->redis->rm($lock_key);
                }
                $lock =  $this->redis->setNX($lock_key , time());
                if($lock){
                    $this->redis->expire($lock_key,  $expire); //如果没执行完 2s锁失效
                }
            }
            return $lock;
        }
        /**
         *  解锁
         */
        public function task_unlock($taskid){
            $this->set_redis();
            $lock_key = 'task_get_reward_'.$this->uid.'_'.$taskid;
            $this->redis->rm($lock_key);
        }
说明下setNX 和expire 这两个操作其实可以用redis事务来保证一致性
