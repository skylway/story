## redis常用监控命令


1.实时监控redis服务收到来自应用的所有命令

　　
```
redis-cli
 
127.0.0.1:6379>monitor
 
1509964152.131699 [1 127.0.0.1:40008] "hmget" "DEFAULEGYM_PK_PLAYER_PROPERTY_10105" "cup" "type" "pkScore" "matchTimeIntervals"
 
1509964152.131699 [1 127.0.0.1:40008] "hmget" "DEFAULEGYM_PK_PLAYER_PROPERTY_10105" "cup" "type" "pkScore" "matchTimeIntervals"

```
执行该命令将会把redis日志全部打印出来，有时间，来源ip，来源端口，操作函数，操作key。我们可以基于这些日志对当前redis使用情况进行统计分析

 

2.查看redis慢日志

　　　　
```
redis-cli

127.0.0.1:6379>slowlog get 128  // 只存储128条满日志，多了会顶掉

1)  1) (integer) 77            // 编号
    2) (integer) 1509876448    // 时间戳
    3) (integer) 28599　　　　　 // 耗时，微妙
    4) 1) "info"　　　　　　　　　// 命令
       2) "loglevel"　　　　　　 // 操作key
 2) 1) (integer) 76
    2) (integer) 1509503373
    3) (integer) 42481
    4) 1) "LPOP"
       2) "WECHATAPP:MESSAGE_LIST_user:ALL"
 
```
该命令把耗时较长的命令列出来，对存取优化很有帮助。

 

3.查看redis服务的各项状态

```
redis-cli <br>127.0.0.1:6379> info

127.0.0.1:6379> info CPU        // cpu使用情况

127.0.0.1:6379> info Keyspace   // 各个db的key的状况，是否有设置超时时间。这是一个很重要的查看项。<br>127.0.0.1:6379> info Stats　　　 // 服务状态<br>...
　　
```
该命令用来查看redis概览各项情况。

 

--------------------------------------------

redis性能查看与监控常用工具

1.redis-benchmark 

redis基准信息，redis服务器性能检测 
redis-benchmark -h localhost -p 6379 -c 100 -n 100000 
100个并发连接，100000个请求，检测host为localhost 端口为6379的redis服务器性能 

