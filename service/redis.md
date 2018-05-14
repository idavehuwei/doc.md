#Redis基础以及主从sentinel

##redis基础
###特性
- redis是基于键值存储(key-value)的noSQL高性能存储
- 通memcache类似都是数据存储在内存,但提供了RDB和AOF两种持久化方式,并在此基础上实现主从(master-slave),高可用(sentinel),集群(Redis Cluster).
- 支持丰富的数据类型string,list,set,hashs,bitmap,hyperloglog,在最新的4.0中加入了stream类型

###安装配置
redis的安装配置非常简单,获得程序包几乎不用配置就可以正常运行.这里对配置文件的配置项做下说明


```
#基本项
include /path/to/other.conf			#引用配置文件。
daemonize yes						#是否在后台执行
protected-mode yes 					#3.2里的参数，是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。要是开启了密码   和bind，可以开启。否   则最好关闭，设置为no。
pidfile /var/run/redis/redis-server.pid 			#进程文件
port 6379							#监听的端口号
tcp-backlog 511						#此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。该内核参数默认值一般是128，对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。在/etc/sysctl.conf中添加:net.core.somaxconn = 2048，然后在终端中执行sysctl -p。
bind 127.0.0.1						#监听IP,不设置默认所有IP
unixsocket /var/run/redis/redis.sock		#unix socket监听文件
unixsocketperm 700				#unix socket使用文件的权限
timeout 0							#空闲超时时间
tcp-keepalive 0					#tcp keepalive参数。如果设置不为0，就使用配置tcp的SO_KEEPALIVE值，使用keepalive有两个好处:检测挂掉的对端。降低中间设备出问题而导致网络看似连接却已经与对端端口的问题。在Linux内核中，设置了keepalive，redis会定时给对端发送ack。检测到对端关闭需要两倍的设置值。

loglevel notice					#日志级别
logfile /var/log/redis/redis-server.log		#指定日志文件
syslog-enabled no				#打开系统syslog功能
syslog-ident redis				#syslog的标识符
syslog-facility local0			#日志的来源、设备

databases 16					#数据库channel

```
```
# rdb快照
# 不使用可以直接save ""
# 900秒（15分钟）内至少1个key值改变进行持久化
# 300秒（5分钟）内至少10个key值改变进行持久化
# 60秒（1分钟）内至少10000个key值改变进行持久化
save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes		#rdb失败是否继续工作yes停止
rdbcompression yes					#rdb是否使用压缩
rdbchecksum yes						#rdb是否校验
dbfilename dump.rdb					#rdb文件名
dir /var/lib/redis					#定义rdb路径

```
```
#复制和主从
slaveof <masterip> <masterport>   	#从节点配置项
masterauth <master-password>		#主节点开启密码,从节点配置密码
slave-serve-stale-data yes			#从节点能否脱离主节点yes(默认设置),从库会继续响应客户端的请求.否则除去INFO和SLAVOF命令之外的任何请求都会返回一个错误
slave-read-only yes					#从节点只读
repl-diskless-sync no				#是否使用socket方式复制数据。目前redis复制提供两种方式，disk和socket。如果新的slave连上来或者重连的slave无法部分同步，就会执行全量同步，master会生成rdb文件。有2种方式：disk方式是master创建一个新的进程把rdb文件保存到磁盘，再把磁盘上的rdb文件传递给slave。socket是master创建一个新的进程，直接把rdb文件以socket的方式发给slave。disk方式的时候，当一个rdb保存的过程中，多个slave都能共享这个rdb文件。socket的方式就的一个个slave顺序复制。在磁盘速度缓慢，网速快的情况下推荐用socket方式。

repl-diskless-sync-delay 5  		#diskless复制的延迟时间，防止设置为0
repl-ping-slave-period 10			#探测时间
repl-timeout 60						#复制超时时间,应该币ping大
repl-disable-tcp-nodelay no			#复制时允许tcp_nodelay默认no延迟小,数据量大建议yes
repl-backlog-size 5mb				#复制缓冲区大小默认1mb
repl-backlog-ttl 3600				#没有复制链接释放缓冲区延迟时间

slave-priority 100					#sentinel优先级.0永远不会被选举
min-slaves-to-write 3				#设置master少于多少个slave时就不允许写入
min-slaves-max-lag 10				#健康的slave延迟小于n秒 0禁用

```
```
#安全
requirepass foobared				#设置密码
rename-command CONFIG ""			#设置成一个空的值，可以禁止一个命令

```
```
#内存相关

maxclients 10000					#默认允许连接数,注意redis不区分客户端,slave,内部打开文件句柄等连接.
maxmemory <bytes>					#最大内存

#内存容量超过maxmemory后的处理策略。
maxmemory-policy noeviction
#volatile-lru：						#利用LRU算法移除设置过过期时间的key。
#volatile-random：					#随机移除设置过过期时间的key。
#volatile-ttl：						#移除即将过期的key，根据最近过期时间来删除（辅以TTL）
#allkeys-lru：						#利用LRU算法移除任何key。
#allkeys-random：						#随机移除任何key。
#noeviction：							#不移除任何key，只是返回一个写错误。

maxmemory-samples 5					#lru检测的样本数。使用lru或者ttl淘汰算法，从样本中选择空闲最久的移除
```
```

#比RDB更可靠的aof持久化
appendonly no
appendfilename "appendonly.aof"		#aof文件名
appendfsync everysec				#aof持久化策略的配置
#no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快。
#always表示每次写入都执行fsync，以保证数据同步到磁盘。
#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据。

no-appendfsync-on-rewrite no		#默认为no,fsync策略30秒,设置aof的阻塞.防止AOF写入IO占用,导致写入失败
auto-aof-rewrite-percentage 100		#AOF日志改变达到多少百分比时重写AOF文件,调用bgrewriteao
auto-aof-rewrite-min-size 64mb		#设置最少重写文件大小,以免文件过小
aof-load-truncated yes				#设置AOF文件因为意外宕机造成文件尾部不完整导致错误,手动redis-check-aof

```
```
lua-time-limit 5000					#lua相关没搞懂
```
```
#集群

cluster-enabled yes					#集群开关
cluster-config-file nodes-6379.conf	#集群节点配置文件
cluster-node-timeout 15000			#节点超时时间
cluster-slave-validity-factor 10	#节点提升为master参数
cluster-migration-barrier 1			#master附属从节点的迁移
cluster-require-full-coverage yes	#检查slot是否完全分配

```
```
#slowlog
slowlog-log-slower-than 10000		#slowlog时间,单位微秒,负数禁用,0记录所有.记录在内存中
slowlog-max-len 128					#slowlog记录的命令长度

latency-monitor-threshold 0			#延迟监控,0默认关闭
```
```
#其它

notify-keyspace-events ""			#查看官网http://redis.io/topics/notifications



hash-max-ziplist-entries 512		#hash数据量小于值使用ziplist大于使用hash
hash-max-ziplist-value 64			#value小于值使用ziplist大于使用hash

list-max-ziplist-entries 512		#list数据量小于值使用ziplist大于使用list
list-max-ziplist-value 64			#value小于值使用ziplist大于使用list

set-max-intset-entries 512			#list数据量小于值使用iniset大于使用set

zset-max-ziplist-entries 128		#zset数据量小于值使用ziplist大于使用zset
zset-max-ziplist-value 64			#value小于值使用ziplist大于使用zset

hll-sparse-max-bytes 3000			#hl建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右。

activerehashing yes					#设置是否自动重新hash,no可以提供实时性.yes占用内存小

client-output-buffer-limit normal 0 0 0			#客户端buffer,第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal client默认取消限制

client-output-buffer-limit slave 256mb 64mb 60	#限制slave链接	一旦超过256mb，又或者超过64mb持续60秒断开slave或moniter连接

client-output-buffer-limit pubsub 32mb 8mb 60	#pubsub缓存一旦超过32mb，又或者超过8mb持续60秒断开连接


hz 10								#redis执行任务的频率为1s除以hz。

aof-rewrite-incremental-fsync yes	#aof fsync开启
```


##redis主从和sentinel
区别于分布式集群,早期redis提供了高可用的主从功能,并且通过sentinel来实现监控,自动发现,和故障转移.

###sentinel特性

- 当master节点不可用是,能够在众多slave中选取一个提升为新的master,并且将其它的slave追随的master地址改为新的节点地址

- 能够监控master-slave的运行情况并通过API通知系统管理员或其它进程.

- 能够根据当前配置去纠正master-slave的配置错误,包括故障转移时的配置修正

- sentinel支持API,自动发现和消息订阅.

- 为了避免单点故障,sentinel也有必要运行为集群模式




###sentinel配置安装
1, 首先启动master-slave,一主2从

> 生成配置文件
```
mkdir /opt/rs/700{1,2,3}/
echo /opt/rs/700{1,2,3} |xargs -n 1 cp /etc/redis.conf
```
> 主从分别配置

```
#master
daemonize yes
port 7001
```
```
#slave
daemonize yes
port 7002
slaveof 127.0.0.1 7001
slave-read-only yes
```

> 启动主从
```
redis-server /opt/rs/7001/redis.conf
...
```
> 检查信息

```
redis-cli -p 7001
127.0.0.1:7001> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=7002,state=online,offset=337,lag=0
slave1:ip=127.0.0.1,port=7003,state=online,offset=337,lag=0
...
```

2, 启动sentinel集群

> 生成配置文件

```
mkdir /opt/sentinel/800{1,2,3} -p
echo /opt/sentinel/800{1,2,3} |xargs -n 1 cp /etc/redis-sentinel.conf
```
> 配置文件修改

```
daemonize yes
port 8001
sentinel monitor mymaster 127.0.0.1 7001 2  #2表示在几个sentinel检测到故障时进行转移
```

> 启动sentinel查看信息

```
redis-sentinel /opt/sentinel/8001/redis-sentinel.conf
...
```
```
redis-cli -p 8001
127.0.0.1:8001> info
...
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:7001,slaves=2,sentinels=3
```
这里我们可以观察到sentinel会自动发现监视同一服务器的其它sentinel

3, sentinel API常用命令
> 查看master信息

```
127.0.0.1:8001> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "127.0.0.1"
...
```

> 故障测试

```
redis-cli -p 7001 debug sleep 300
```

```
127.0.0.1:8001> sentinel get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "7003"
#这是发现master已经转移到7003了
```

> 修改参与仲裁的最低数量

```
sentinel set mymaster quorum 1
```
```
127.0.0.1:8002> sentinel masters
1)  1) "name"
    2) "mymaster"
...
   35) "quorum"
   36) "1"
   37) "failover-timeout"
   38) "180000"
   39) "parallel-syncs"
   40) "1"
```

> 命令列表

```
SENTINEL masters 					#查看所有master
SENTINEL master <master name> 		#查看master实例
SENTINEL slaves <master name>		#查看master实例的从属
SENTINEL sentinels <master name> 	#查看master的sentinels(在sentinel上查询结果是不包括当前实例)
SENTINEL get-master-addr-by-name 	#查看master的ip地址
SENTINEL reset <pattern>			#重置实例
SENTINEL failover <master name>		#手动故障转移,不需要其它sentinel同意,并且会发送新的master配置
SENTINEL ckquorum <master name> 	#检查sentinel是否正常
SENTINEL flushconfig				#强制将配置文件刷写到硬盘
SENTINEL MONITOR <name> <ip> <port> <quorum>	#添加新的master
SENTINEL REMOVE <name> 							#移除一个监控目标
SENTINEL SET <name> <option> <value> 			#修改sentinel对于master的配置
SENTINEL auth-pass <master-group-name> <pass>	#设置密码
```

4, 添加删除sentinel

> 添加

在官方文档中的说明,sentinel是基于自动发现的.只需要对sentinel配置相同的master就会自动添加.只是需要注意在每个sentinel启动时间隔30秒,以便每个sentinel有一个等待发现的过程

> 删除

sentinel不会主动删除发现过的其它实例信息,即使sentinel长期无法访问.


- 停止希望删除的sentine
- 发送SENTINEL RESET * 并等待在线的sentinel确认
- 去所有在线sentinel检查.

5, 添加master
需要注意的sentinel是以monitor时设置的name来群分sentinel集群,所以不同的name是不通sentinel集群.
当然master的每个sentinel都需要在配置文件或者是命令行中配置sentinel monitor

> 启动额外master
```
echo rs1/900{1,2,3}/redis.conf |xargs -n 1 redis-server
```

> sentinel添加master 
```
redis-cli -p 8001
127.0.0.1:8001> sentinel monitor mymaster2 127.0.0.1 9001 2
...
```

> 查看信息
```
127.0.0.1:8001> sentinel masters
1)  1) "name"
    2) "mymaster2"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "9001"
...
2)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "7003"

```

6, sentinel其它相关
关于API和sentinel消息订阅查看[官方文档](https://redis.io/topics/sentinel)



######总结两个问题:

一:sentinel和master-slave虽然解决了redis的高可用和故障转移.但是并不能良好的解决应用中统一入口的问题,当发生故障转移,master的ip和端口可能产生改变.

二: 由于我们通常将slave设置为只读模式,所以应用的负载均衡功能实现比较复杂.






#####最后

redis中文翻译网站: [地址](http://doc.redisfans.com/)


























