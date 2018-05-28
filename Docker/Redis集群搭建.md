# 主从模式

## 准备配置文件：

```shell
docker pull redis
➜  redis pwd
/Users/borgxiao/docker/redis

#这里使用最新的4.0
wget https://raw.githubusercontent.com/antirez/redis/4.0/redis.conf 
➜  redis ls
redis.conf

#修改配置文件的slaveof
slaveof redis-master 6379
```



## 启动主服务

```Shell
docker run --name redis-master -p 6379:6379 -d redis
```

### 启动2个节点：

```shell
#节点1 redis-slave1
docker run --link redis-master:redis-master -v ~/docker/redis/redis.conf:/usr/local/etc/redis/redis.conf --name redis-slave1 -p 7000:6379 -d redis redis-server /usr/local/etc/redis/redis.conf
#节点2 redis-slave2
docker run --link redis-master:redis-master -v ~/docker/redis/redis.conf:/usr/local/etc/redis/redis.conf --name redis-slave2 -p 7001:6379 -d redis redis-server /usr/local/etc/redis/redis.conf

➜  redis docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED                  STATUS              PORTS                    NAMES
14daf3085b22        redis               "docker-entrypoint.s…"   Less than a second ago   Up 3 seconds        0.0.0.0:7001->6379/tcp   redis-slave2
db4a450f7943        redis               "docker-entrypoint.s…"   About a minute ago       Up About a minute   0.0.0.0:7000->6379/tcp   redis-slave1
692df59f0e42        redis               "docker-entrypoint.s…"   3 minutes ago            Up 3 minutes        0.0.0.0:6379->6379/tcp   redis-master
```



## 验证：

直接本地调用redis-cli命令连接：

```shell
➜  redis redis-cli
127.0.0.1:6379> info
# Server
redis_version:4.0.9
redis_git_sha1:00000000
redis_git_dirty:0
# Replication
role:master
connected_slaves:2
slave0:ip=172.17.0.3,port=6379,state=online,offset=336,lag=0
slave1:ip=172.17.0.4,port=6379,state=online,offset=336,lag=0
master_replid:700d4df4fae0a76a8f22b791c4ab241afd13df87

#主
➜  redis redis-cli
127.0.0.1:6379> set key1 haha
OK
127.0.0.1:6379> get key1
"haha"

#节点1
➜  redis redis-cli -p 7000
127.0.0.1:7000> get key1
Error: Server closed the connection
```

## 修复密码问题：

```shell
#修改redis.conf配置项
protected-mode no //默认为yes
```

修改后重新启动节点，记得先删除，还是不行，原因未知



# 集群模式：

## 本机：

```shell
#准备redis.conf 6份 创建6个目录
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes

➜  redis pwd
/Users/borgxiao/docker/redis
➜  redis ls
7000               7003               redis-trib.rb
7001               7004               redis.cluster.conf
7002               7005               redis.conf

#分别在对应目录执行：启动redis
➜  7005 nohup redis-server redis.conf > /dev/null &

#
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

#查看信息1主1从
➜  redis ./redis-trib.rb info 127.0.0.1:7000
127.0.0.1:7000 (23cb83a9...) -> 0 keys | 5461 slots | 1 slaves.
127.0.0.1:7001 (74e3a67d...) -> 0 keys | 5462 slots | 1 slaves.
127.0.0.1:7002 (ea57a986...) -> 0 keys | 5461 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.

➜  redis ./redis-trib.rb check 127.0.0.1:7004
>>> Performing Cluster Check (using node 127.0.0.1:7004)
S: e443d7a36730d2095ebf8dc2ce00040fae6ae1d7 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 23cb83a958a811d75b7ad7f80681cddac5d6946e
M: ea57a986ddbfa74ea3d7f281dc991a602a40c3a9 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: dca5b7276d357acb62fc89a19ee8e9db392c3f8e 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 74e3a67da4af4d4f389b44d96372d17ef29c78a6
S: 674c955833a744c4cf4dab9f0e296af806fcede6 127.0.0.1:7003
   slots: (0 slots) slave
   replicates ea57a986ddbfa74ea3d7f281dc991a602a40c3a9
M: 74e3a67da4af4d4f389b44d96372d17ef29c78a6 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 23cb83a958a811d75b7ad7f80681cddac5d6946e 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.


127.0.0.1:7000> get key1
(error) MOVED 9189 127.0.0.1:7001

➜  redis redis-cli -p 7000 cluster nodes
e443d7a36730d2095ebf8dc2ce00040fae6ae1d7 127.0.0.1:7004@17004 slave 23cb83a958a811d75b7ad7f80681cddac5d6946e 0 1526998920855 5 connected
74e3a67da4af4d4f389b44d96372d17ef29c78a6 127.0.0.1:7001@17001 master - 0 1526998920339 2 connected 5461-10922
23cb83a958a811d75b7ad7f80681cddac5d6946e 127.0.0.1:7000@17000 myself,master - 0 1526998920000 1 connected 0-5460
dca5b7276d357acb62fc89a19ee8e9db392c3f8e 127.0.0.1:7005@17005 slave 74e3a67da4af4d4f389b44d96372d17ef29c78a6 0 1526998920000 6 connected
674c955833a744c4cf4dab9f0e296af806fcede6 127.0.0.1:7003@17003 slave ea57a986ddbfa74ea3d7f281dc991a602a40c3a9 0 1526998920000 4 connected
ea57a986ddbfa74ea3d7f281dc991a602a40c3a9 127.0.0.1:7002@17002 master - 0 1526998919110 3 connected 10923-16383

#关闭一台
➜  redis redis-cli -p 7002 debug segfault
Error: Server closed the connection
[3]    90035 segmentation fault  nohup redis-server redis.conf > /dev/null
```

## docker下：

```shell
#依次创建
➜  redis docker run --name redis-7005 -p 7005:6379 -v ~/docker/redis/redis.cluster.conf:/usr/local/etc/redis/redis.conf -d redis redis-server /usr/local/etc/redis/redis.conf
103a80d5539676961f1121bfacabcee084ee889b9e1931d8088571a115e411c5
➜  redis docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED                  STATUS              PORTS                              NAMES
103a80d55396        redis               "docker-entrypoint.s…"   Less than a second ago   Up 6 seconds        6379/tcp, 0.0.0.0:7000->7000/tcp   redis-7000

#执行不下去，网络不通
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

#换--net=host 还是不行
docker run --name redis-7000 --net=host -v ~/docker/redis/redis.cluster.conf:/usr/local/etc/redis/redis.conf -d redis redis-server /usr/local/etc/redis/redis.conf
```



## 参看链接：

主从模式：https://segmentfault.com/a/1190000004353368

官网本机集群模式http://www.redis.cn/topics/cluster-tutorial.html

redis-trib.rb错误解决：https://www.cnblogs.com/ytfcz/p/5275633.html

命令redis-trib.rb：https://blog.csdn.net/huwei2003/article/details/50973967 