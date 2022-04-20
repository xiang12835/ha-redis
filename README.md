# hredis
High Availability Redis, Spring Boot

>Redis 主从复制

演示：搭建一主两从

1. 修改redis.conf 配置文件

redis6379.conf（主）, redis6380.conf（从）, redis6381.conf（从）

修改端口号port，文件夹dir，pidifile

2. 启动，使用`slaveof`配置主从

```
$ redis-server redis6379.conf & # 主库
$ redis-server redis6380.conf & # 启动从库1
$ redis-server redis6381.conf & # 启动从库2

$ redis-cli -h 127.0.0.1 -p 6380 # 连接从库1
slaveof 127.0.0.1 6379

$ redis-cli -h 127.0.0.1 -p 6381 # 连接从库2
slaveof 127.0.0.1 6379
```


>Redis Sentinel 自动主从切换 ～ 类似于MySQL的MHA


![](http://110.40.156.239:3000/images/bd42ea1997064efab1931e9c645c3051.png)

Sentinel 在集群里，相当于注册中心。

可以做到监控主从节点的在线状态，并做切换（基于raft协议）。

演示：搭建一主两从两哨兵

- 配置两个 sentinel0.conf, sentinel1.conf
- 启动两个 sentinel 的节点（26379和26380），专门去监控redis-server（6379）的状态和自动主从切换

步骤：

1. sentinel.conf配置：

```
sentinel monitor mymaster 127.0.0.1 6379 2 
sentinel down-after-milliseconds mymaster 60000 
sentinel failover-timeout mymaster 180000 
sentinel parallel-syncs mymaster 1 
```

哨兵只要配置主库，就可以知道主库和从库的信息，所以不要给从库配置。不需要配置从节点，也不需要配置其他sentinel信息。


2. 两种启动方式：

```
$ redis-sentinel sentinel0.conf 

 或者

$ redis-server redis.conf --sentinel 
```

复制 sentinel0.conf，修改端口和ID号，命名为sentinel1.conf


3. 关闭主库

```
$ redis-cli -h 127.0.0.1 -p 6379
shutdown
exit
```

4. 自动切换，使用 info 查看主从信息

另外，可以使用客户端直接连接 Sentinel

```
redis-cli -h 127.0.0.1 -p 26380

>info 

or

>sentinel masters

```

资料：

redis sentinel原理介绍：http://www.redis.cn/topics/sentinel.html

redis复制与高可用配置：https://www.cnblogs.com/itzhouq/p/redis5.html

