

#Redis Cluster 
##官方文档节选

[文档地址](https://redis.io/topics/cluster-tutorial)

1. automatically sharded

> * The ability to automatically split your dataset among multiple nodes.
> * The ability to continue operations when a subset of the nodes are experiencing failures or are unable to communicate with the rest of the cluster.

Redis Cluster集群是以分布式运行支持高达1000个节点,会将数据自动进行分割存储在集群中的不同节点.除非是大部分的master节点宕机或不可访问.

2. TCP ports

> * The normal client communication port (usually 6379) used to communicate with clients to be open to all the clients that need to reach the cluster, plus all the other cluster nodes (that use the client port for keys migrations).
> * The cluster bus port (the client port + 10000) must be reachable from all the other cluster nodes.

Redis Cluster集群服务使用了两种TCP端口,会监听一个普通端口和总线端口(bus port英语没过级不知道怎么翻译好.),普通端口和其它模式下的6379端口一样,用于客户端与集群中节点交互.集群中的总线端口是用于集群节点之间故障检测,配置更新,授权等等,如果总线端口不能通信集群也就不能连接到这个节点.总线端口总是固定在普通端口基础上偏移10000.

3. Hash slot

Redis Cluster集群没有使用一致性hash,而是使用的hash slot(哈希槽).每个集群总共有16384个哈希槽,集群会根据节点的数量分配给每个主节点一个哈希槽子集(即一部分哈希槽).因此可以动态增加和删除节点,只需要重新分配哈希槽.同时启用了一个称为hash tags的概念,因此集群模式下只能使用database 0.(hash tags不懂...)

4. master-slave model

Redis Cluster集群也支持主从模式,必须保证主节点必须要有对应的一个或多个从节点,在主节点不能访问的时候,其中一个从节点会提升为主节点.否则集群在主节点宕机后将无法工作.

5. consistency guarantees

Redis Cluster不能保证节点间良好的一致性,所以在某些极端情况下会出现数据不一致的情况.因为主从之间使用的异步写入,并且不会等待确认返回.如果需要使用同步写入的话可以使用WAIT命令.即使是同步写入也有不能写入的从节点被提升为主节点造成的写入失败的情况发生.(官方举的这个例子和我遇到的一次mysql主备切换造成binlog丢失的情况很像,有一次生产环境的mysql主库使用率100%持续很久,造成切换备用节点.然后这些数据只能在被切的那个机器上找到.其它的备库,从库通通没有binlog日志.)

##集群配置

1. 集群安装配置

```
#Redis Cluster集群管理工具
yum install redis-trib -y
#gem install redis
```

- 创建配置文件及配置参数  

> * cluster-enabled <yes/no>: 				#集群是否启用  
> * cluster-config-file <filename>:  		#这个不是用户需要编辑的配置文件,而是集群在运行中所采集到所有的节点信息.一遍在集群重启时能够正确的自动配置集群信息   
> * cluster-node-timeout <milliseconds>: 	#超时时间,超过这个时间主节点将被它的从节点取代.需要注意的是:任何节点在指定时间不能连接大多数主节点,将停止接受查询.   
> * cluster-slave-validity-factor <factor>: #不建议设置,没搞懂的话,在故障恢复时会造成苦难   
> * cluster-migration-barrier <count>: 		#副本迁移相关   
> * cluster-require-full-coverage <yes/no>: #节点通信密钥相关   

- 配置节点实例

```
mkdir /etc/redis.d/700{1,2,3,4,5,6} -p
echo /etc/redis.d/700{1,2,3,4,5,6} |xargs -n 1 cp -v /etc/redis.conf
```

```
#将每个节点的端口和cluster-conifg-file修改为不同
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

```
redis-server /etc/redis.d/7001/redis.conf
...
redis-server /etc/redis.d/7006/redis.conf
```

- 创建集群

执行创建集群的命令create ,replicas 1表示每个主节点分配一个从节点

```
[root@localhost ~]# redis-trib create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
```

2. 增加和删除节点


- 增加节点

> 查看节点信息
```
redis-cli -p 30001 cluster nodes
a2e26df91d65847a17f4172b0b61f840b0948c56 127.0.0.1:30003@40003 master - 0 1516419900548 3 connected 10923-16383
...
```

> 启动实例主节点30007.从节点30008,
```
redis-server --port 30007 --cluster-enabled yes --cluster-config-file nodes-30007.conf --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30007.aof --dbfilename dump-30007.rdb --logfile 30007.log --daemonize yes
...
```
> 加入集群
```
redis-trib add-node 127.0.0.1:30007 127.0.0.1:30001
redis-trib  add-node --slave --master-id d0617fb34ef85de4e414b62c4a82f8b3e0aa2e1f 127.0.0.1:30008 127.0.0.1:30001
```
> 查看节点信息,此时新加入的节点没有分配hash slot所以是一个空节点并不参入集群的工作
```
redis-cli -c -p 30001 cluster nodes
...
65537824bef847f3f199f9b076a928d860f00c7a 127.0.0.1:30008@40008 slave d0617fb34ef85de4e414b62c4a82f8b3e0aa2e1f 0 1516423427008 0 connected
d0617fb34ef85de4e414b62c4a82f8b3e0aa2e1f 127.0.0.1:30007@40007 master - 0 1516421726020 0 connected
...
```
> reshared重新分配hash slot

```
redis-trib reshard 127.0.0.1:30001 --slots 4096 --from all --to d0617fb34ef85de4e414b62c4a82f8b3e0aa2e1f --yes 127.0.0.1:30001
```
```
redis-cli -c -p 30001 cluster nodes
...
d0617fb34ef85de4e414b62c4a82f8b3e0aa2e1f 127.0.0.1:30007@40007 master - 0 1516423765000 8 connected 0-1364 5461-6826 10923-12287
#此时可以看到30007的slots是从其它三个主节点上平均分配了过来
```

> 故障测试,从节点会自动提升为主机点

```
redis-cli -p 30007 debug segfault
```
```
redis-cli -c -p 30001 cluster nodes
...
65537824bef847f3f199f9b076a928d860f00c7a 127.0.0.1:30008@40008 master - 0 1516424352174 9 connected 0-1364 5461-6826 10923-12287
d0617fb34ef85de4e414b62c4a82f8b3e0aa2e1f 127.0.0.1:30007@40007 master,fail - 1516424346060 1516424345046 8 disconnected
...
# 在重启故障节点后,30007会自动加入为30008的从节点
```

> 删除节点

```
redis-trib reshard 127.0.0.1:30001 --slots 4096 --from 65537824bef847f3f199f9b076a928d860f00c7a --to 3dda66a50f8b1f4601a94be6b7e110c04501017d --yes
redis-trib info 127.0.0.1:30007
127.0.0.1:30008@40008 (65537824...) -> 0 keys | 0 slots | 0 slaves.
127.0.0.1:30002@40002 (3dda66a5...) -> 98 keys | 8192 slots | 2 slaves
...
#此时30008的从节点也会自动归属到新的主节点上去
```
```
redis-trib del-node 127.0.0.1:30001 65537824bef847f3f199f9b076a928d860f00c7a
```

> 重新平衡slots

```
redis-trib rebalance --auto-weights 127.0.0.1:30001
```
```
redis-trib info 127.0.0.1:30001
127.0.0.1:30001 (09a3e281...) -> 68 keys | 5462 slots | 1 slaves.
127.0.0.1:30003@40003 (a2e26df9...) -> 67 keys | 5461 slots | 1 slaves.
127.0.0.1:30002@40002 (3dda66a5...) -> 63 keys | 5461 slots | 2 slaves.
```



##集群迁移

做集群迁移的时候官方已经提供了很多意见.
```
1. Stop your clients. No automatic live-migration to Redis Cluster is currently possible. You may be able to do it orchestrating a live migration in the context of your application / environment.

2. Generate an append only file for all of your N masters using the BGREWRITEAOF command, and waiting for the AOF file to be completely generated.

3. Save your AOF files from aof-1 to aof-N somewhere. At this point you can stop your old instances if you wish (this is useful since in non-virtualized deployments you often need to reuse the same computers).

4. Create a Redis Cluster composed of N masters and zero slaves. You'll add slaves later. Make sure all your nodes are using the append only file for persistence.

5. Stop all the cluster nodes, substitute their append only file with your pre-existing append only files, aof-1 for the first node, aof-2 for the second node, up to aof-N.

6. Restart your Redis Cluster nodes with the new AOF files. They'll complain that there are keys that should not be there according to their configuration.

7. Use redis-trib fix command in order to fix the cluster so that keys will be migrated according to the hash slots each node is authoritative or not.

8. Use redis-trib check at the end to make sure your cluster is ok.

9. Restart your clients modified to use a Redis Cluster aware client library.
```

1.    保存aof文件并复制. 

```
redis-trib call 127.0.0.1:30001 BGREWRITEAOF
```
```
mkdir /opt/redis-test

...
```
2. 启动新的集群节点.

```
#修改脚本
vi redis-cluster.sh
PORT=10000
TIMEOUT=2000
NODES=3
REPLICAS=0
...
./redis-cluster.sh start
....
```
3. 使用redis-cli配置集群(使用redis的原生命令)

> 分配slots
```
redis-cli -c -p 10001 cluster addslots {0..5460}
redis-cli -c -p 10002 cluster addslots {5461..10922}
redis-cli -c -p 10003 cluster addslots {10922..16383}
```
> 设置epoch
```
redis-cli -c -p 10001 cluster set-config-epoch 1
redis-cli -c -p 10002 cluster set-config-epoch 2
redis-cli -c -p 10003 cluster set-config-epoch 3
```
> 加入集群
```
redis-cli -c -p 10003 cluster meet 127.0.0.1 10001
redis-cli -c -p 10003 cluster meet 127.0.0.1 10002
```

```
redis-trib check 127.0.0.1:10001
>>> Performing Cluster Check (using node 127.0.0.1:10001)
M: d5a115418f657f4b25229af38b5b88036c90b3b4 127.0.0.1:10001
   slots:0-5460 (5461 slots) master
   0 additional replica(s)
```
4. 复制aof文件

```
./redis-cluster.sh stop
cp ~/appendonly-30001.aof ./appendonly-10001.aof
...
./redis-cluster.sh start
```

5. 修复集群 

```
redis-trib fix 127.0.0.1:10001
redis-trib check 127.0.0.1:10001
```

6. 查看数据

```
redis-test]# redis-cli -c -p 10001
127.0.0.1:10001> keys *
 1) "foo96"
 2) "foo56"
 3) "foo67"
```
##集群管理工具命令参数

1. redis-trib子命令

```
  create          host1:port1 ... hostN:portN  #创建集群
                  --replicas <arg>	#表示每个主节点的从节点数量
  check           host:port   		#检查节点
  info            host:port	  		#输出主节点信息 keys,slots,slaves
  fix             host:port   		#修复集群,比较迁移后配置问题
                  --timeout <arg>
  reshard         host:port   		#分配slots
                  --from <arg>  	#源节点
                  --to <arg>		#目标节点
                  --slots <arg>		#slots数量
                  --yes				#不使用交互模式
                  --timeout <arg>	
                  --pipeline <arg>	#定义每次cluster getkeysinslot取值,默认10
  rebalance       host:port			#平衡slots	
                  --weight <arg>	#节点权重,多个节点指定多个
                  --auto-weights	#自动分配权重
                  --use-empty-masters	#是否启用空节点参与
                  --timeout <arg>
                  --simulate		#模拟操作
                  --pipeline <arg>
                  --threshold <arg>	#设置slots平衡算法参数
  add-node        new_host:new_port existing_host:existing_port
                  --slave 			#新节点为从节点
                  --master-id <arg> #做为从节点复制的主节点ID
  del-node        host:port node_id
  set-timeout     host:port milliseconds
  call            host:port command arg arg .. arg	#执行redis-cli命令
  import          host:port			#导入数据
                  --from <arg>
                  --copy
                  --replace
  help            (show this help)
```

2. redis原生集群命令

```
cluster info       				#打印集群的信息
cluster nodes   				#列出集群当前已知的所有节点(node)，以及这些节点的相关信息   
节点(node)  
cluster meet <ip> <port>        #将ip和port所指定的节点添加到集群当中，让它成为集群的一份子  
cluster forget <node_id>        #从集群中移除node_id指定的节点
cluster replicate <node_id>   	#将当前节点设置为node_id指定的节点的从节点
cluster saveconfig              #将节点的配置文件保存到硬盘里面
cluster slaves <node_id>       	#列出该slave节点的master节点
cluster set-config-epoch        #强制设置configEpoch 
```
> 槽(slot)  
```
cluster addslots <slot> [slot ...]               #将一个或多个槽(slot)指派(assign)给当前节点
cluster delslots <slot> [slot ...]               #移除一个或多个槽对当前节点的指派 
cluster flushslots              				 #移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点 
cluster setslot <slot> node <node_id>            #将槽slot指派给node_id指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽，然后再进行指派 
cluster setslot <slot> migrating <node_id>       #将本节点的槽slot迁移到node_id指定的节点中  
cluster setslot <slot> importing <node_id>       #从node_id 指定的节点中导入槽slot到本节点 
cluster setslot <slot> stable                    #取消对槽slot的导入(import)或者迁移(migrate) 
```
> 键(key)  
```
cluster keyslot <key>                            #计算键key应该被放置在哪个槽上  
cluster countkeysinslot <slot>                   #返回槽slot目前包含的键值对数量 
cluster getkeysinslot <slot> <count>             #返回count个slot槽中的键
```
> 其它
```
cluster myid                   #返回节点的ID
cluster slots                  #返回节点负责的slot
cluster reset     			   #重置集群，慎用
```


3. 启动脚本
 
> ```
> #!/bin/bash
> 
> # Settings
> PORT=30000
> TIMEOUT=2000
> NODES=6
> REPLICAS=1
> 
> # You may want to put the above config parameters into config.sh in order to
> # override the defaults without modifying this script.
> 
> if [ -a config.sh ]
> then
>     source "config.sh"
> fi
> 
> # Computed vars
> ENDPORT=$((PORT+NODES))
> 
> if [ "$1" == "start" ]
> then
>     while [ $((PORT < ENDPORT)) != "0" ]; do
>         PORT=$((PORT+1))
>         echo "Starting $PORT"
>         redis-server --port $PORT --cluster-enabled yes --cluster-config-file nodes-${PORT}.conf --cluster-node-timeout $TIMEOUT --appendonly yes --appendfilename appendonly-${PORT}.aof --dbfilename dump-${PORT}.rdb --logfile ${PORT}.log --daemonize yes
>     done
>     exit 0
> fi
> 
> if [ "$1" == "create" ]
> then
>     HOSTS=""
>     while [ $((PORT < ENDPORT)) != "0" ]; do
>         PORT=$((PORT+1))
>         HOSTS="$HOSTS 127.0.0.1:$PORT"
>     done
>     redis-trib create --replicas $REPLICAS $HOSTS
>     exit 0
> fi
> 
> if [ "$1" == "stop" ]
> then
>     while [ $((PORT < ENDPORT)) != "0" ]; do
>         PORT=$((PORT+1))
>         echo "Stopping $PORT"
>         redis-cli -p $PORT shutdown nosave
>     done
>     exit 0
> fi
> 
> if [ "$1" == "watch" ]
> then
>     PORT=$((PORT+1))
>     while [ 1 ]; do
>         clear
>         date
>         redis-cli -p $PORT cluster nodes | head -30
>         sleep 1
>     done
>     exit 0
> fi
> 
> if [ "$1" == "tail" ]
> then
>     INSTANCE=$2
>     PORT=$((PORT+INSTANCE))
>     tail -f ${PORT}.log
>     exit 0
> fi
> 
> if [ "$1" == "call" ]
> then
>     while [ $((PORT < ENDPORT)) != "0" ]; do
>         PORT=$((PORT+1))
>         redis-cli -p $PORT $2 $3 $4 $5 $6 $7 $8 $9
>     done
>     exit 0
> fi
> 
> if [ "$1" == "clean" ]
> then
>     rm -rf *.log
>     rm -rf appendonly*.aof
>     rm -rf dump*.rdb
>    rm -rf nodes*.conf
>     exit 0
> fi
> 
> if [ "$1" == "clean-logs" ]
> then
>     rm -rf *.log
>     exit 0
> fi
> 
> echo "Usage: $0 [start|create|stop|watch|tail|clean]"
> echo "start       -- Launch Redis Cluster instances."
> echo "create      -- Create a cluster using redis-trib create."
> echo "stop        -- Stop Redis Cluster instances."
> echo "watch       -- Show CLUSTER NODES output (first 30 lines) of first > node."
> echo "tail <id>   -- Run tail -f of instance at base port + ID."
> echo "clean       -- Remove all instances data, logs, configs."
> echo "clean-logs  -- Remove just instances logs."
> ```



