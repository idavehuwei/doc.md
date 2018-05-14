#systemd

官方文档: [地址](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)

systemd是RH7系列操作系统开始启用新的系统和服务管理器。它被设计为与sysv init脚本向后兼容，并提供了一些功能，例如在引导时并行启动系统服务，按需激活守护程序或基于依赖关系的服务控制逻辑。

systemd中引入了system units的概念,在units其中封装有关系统服务(service),侦听套接字(socket),以及与init系统启动相关信息

###systemd unit 类型
```
unit类型		|	文件后缀名	|	描述
Service unit	|	.service	|	服务类
Target unit		|	.target		|	一个unit服务组,用于模拟实现运行级别
Automount unit	|  .automount	|	文件系统自动挂载点
Device unit		|	.device		|	内核识别的设备文件
Mount unit		|	.mount		|	文件系统挂载点
Path unit		|	.path		|	文件或目录
Scope unit		|	.scope		|	外部创建的进程
Slice unit		|	.slice		|	A group of hierarchically organized units that manage system processes.
Snapshot unit	|	.snapshot	|	系统快照
Socket unit		|	.socket		|	套接字
Swap unit		|	.swap		|	标识swap设备
Timer unit		|	.timer		|	systemd的计时器
```

###unit文件保存位置

```
/usr/lib/systemd/system/	|	RPM包安装时分发的unit文件
/run/systemd/system/		|	systemd运行时创建的文件
/etc/systemd/system/		|	systemctl enable创建的unit文件
```

###主要特性

- 基于socket的激活机制: 在程序不可用时,socket会保持可访问状态,并且所有消息都排队等待.实现了socket与服务程序分离,可以为服务启动一个socket而不需要立即启动程序,通过socket连接激活程序.并以此实现了服务的并行启动.

- 基于bus的激活机制: 使用d-bus进行进程间通信的系统服务可以在客户端应用程序第一次尝试与它们进行通信时按需启动

- 基于device的激活机制: 使用d-bus进行进程间通信的系统服务可以在客户端应用程序第一次尝试与它们进行通信时按需启动.利用USB,CD等设备挂载时激活服务

- 基于path的激活机制: 支持基于路径的激活的系统服务可以在特定文件或目录更改其状态时按需启动.

- 基于mount的激活机制: systemd监视和管理挂载点或自动挂载点激活服务

- 并行启动系统服务: 根据socket激活机制,只要服务所需要的侦听套接字就位,systemd就可以并行启动系统服务.减少系统启动时间

- 系统快照: 保存unit的当前状态于持久设备中,必要时载入.例如重启前保存unit状态,重启后不重新初始化服务直接使用保存的状态.

- 激活逻辑: 在激活或关闭单元之前，systemd会计算它的依赖关系,创建一个临时事务,并验证这个事务是否一致.
如果事务不一致,systemd将自动尝试纠正错误,并在报告错误之前从中删除不重要的作业.

- 兼容sysv init: 支持sysv init风格的启动脚本

###兼容性
systemd系统和服务管理器的设计主要是兼容sysv init和upstart.主要的兼容性变化包括:

- systemd对运行级别的支持有限.对早期的0-6的运行级别概念,systemd是模拟实现的,并不能一一对应的实现,但并不是所有的systemd目标都可以直接映射到运行级别,因此,runlevel这个命令可能会返回n来指示未知的运行级别.同时避免使用init3,5来切换运行级别.

- systemctl子命令是预定义好的,不支持自定义命令.对于centos7以前编写sysv init启动脚本除了定义start,stop,status等命令以外,我们还可以在脚本中自定义其它的子命令.在systemd中不能实现

- 不能控制使用systemctl管理工具以外启动的服务,因为使用systemctl启动服务.systemctl会存储服务的元信息来查询和管理服务.所以通过命令行启动的守护进程,systemd是无法确定服务状态.

- systemd会检查服务运行状态,所以在关机时只会停止正在运行的服务进程

- systemd不会读取任何标准输入数据流

- systemd不会从调用用户及其会话继承任何上下文(如主目录和环境变量),所以编写服务时都使用绝对路径

- 5分钟超时时间,任何服务如果没有被正常执行都会有5分钟超时时间限制.

###管理系统服务

在以前版本使用sysv init或upstart管理位于/etc/rc.d/init.d/目录中的脚本.在centos7中被service unit取代.是用systemctl命令管理

1, 服务基本命令对比

> service和systemctl
```
service							|	systemctl									|	
service name start				|	systemctl start name.service				|	启动服务
service name stop				|	systemctl stop name.service					|	停止服务
service name restart			|	systemctl restart name.service				|	重启服务(没启动的服务会启动)
service name condrestart		|	systemctl try-restart name.service			|	只重启正在运行的服务
service name reload				|	systemctl reload name.service				|	重载配置文件
service name status				|	systemctl status name.service				|	检查服务状态
service name status				|	systemctl is-active name.service			|	检查服务是否启动
service --status-all			|	systemctl list-units --type service --all	|	显示所有的服务状态
```

> chkconfig和systemctl

```
chkconfig				|	systemctl									|	
chkconfig name on		|	systemctl enable name.service 				|	启用开机自启服务
chkconfig name off		|	systemctl disable name.service				|	停用自启服务
chkconfig --list name	|	systemctl status name.service				|	检查服务状态
chkconfig --list name	|	systemctl is-enabled name.service			|	查看服务是否自启
chkconfig --list		|	systemctl list-unit-files --type service	|	查看所有服务
chkconfig --list		|	systemctl list-dependencies --after			|	列出在指定服务之前启动的服务.(依赖)
chkconfig --list		|	systemctl list-dependencies --before		|	列出在指定服务之后启动的服务.(被依赖)
```

2, 服务状态信息

```
Loaded		|	关于服务是否已经加载的信息，文件的绝对路径以及是否被启用的注释。
Active		|	服务是否正在运行,然后是启动时间信息
Main PID	|	服务主进程pid
Docs		|	服务的帮助文档(man)
Status		|	系统服务的额外信息
Process		|	进程额外信息
CGroup		|	Control Groups额外信息
```

3, 常用命令(start,stop,这些常用命令就不说明了)


> 列出所有当前激活服务
```
systemctl list-units --type service
```

> 列出所有服务,不管是否激活(LOAD为notfound应该是还没有安装)

```
systemctl list-units --type service --all
```

> 列出可开机自启的服务

```
systemctl list-unit-files --type service
```

> 重新加载服务:  一个服务设置为开机启动使用enable会将/usr/lib/systemd/system/name.service软链接到/etc/systemd/system/.但是enable命令不会重写已经存在的链接,所以当我们修改了服务文件就需要重新加载

```
systemctl reenable name.service
```

> 禁用服务 mask会将 /etc/systemd/system/name.service软链接到/dev/null.从而禁止服务启用.反操作unmask

```
systemctl mask name.service
```

> 显示服务属性信息

```
systemctl show auditd
```
> 服务的依赖关系

```
systemctl list-dependencies chronyd.service
```
>结束服务

```
systemctl kill name.service
```

###Target

在Centos7之前版本中,拥有0-6编号的一组运行级别代表特定的操作模式.在Centos7中由systemd的target取代,通过一系列依赖关系将其他systemd units组合在一起,来模拟一个运行级别的概念.在运行级别的基础上target有更丰富更灵活的运行模式.

1,sysv和systemd 区别


```
Runlevel|	Target Units							|
	0	|	runlevel0.target, poweroff.target		|	关机
	1	|	runlevel1.target, rescue.target			|	单用户,救援模式
	2	|	runlevel2.target, multi-user.target		|	多用户,非完全启动的命令行(比如网络)
	3	|	runlevel3.target, multi-user.target		|	建立了一个非图形化多用户系统
	4	|	runlevel4.target, multi-user.target		|	预留,未启用
	5	|	runlevel5.target, graphical.target		|	图形界面
	6	|	runlevel6.target, reboot.target			|	重启
```
```
	sysv		|		systemd							|
	runlevel	|	systemctl list-units --type target	|	查看运行级别.
	init N		|	systemctl isolate name.target		|	改变运行级别
```

2, 常用命令

> 获取默认target

```
systemctl get-default
```

> 查看激活的target

```
systemctl list-units --type target
```
> 查看所有target

```
systemctl list-units --type target --all
```

> 修改默认target

```
systemctl set-default name.target
```

> 切换级别

```
systemctl isolate graphical.target
```

> 救援模式

```
systemctl rescue
#会加载驱动
```
```
systemctl emergency
#emergency模式不会加载驱动和系统服务
```

###关机,重启相关

> 关机
```
systemctl halt 
systemctl poweroff
```
> 重启
```
systemctl reboot
```
> 挂起
将系统状态保存在内存中,并关闭大部分设备.(不推荐,容易断电导致内存中数据丢失)

```
systemctl suspend
```

> 休眠
将系统状态保存在硬盘中并关闭.下次启动直接充保存的文件中读取系统信息
```
systemctl hibenate
```
> 休眠并挂起

```
systemctl hybrid-sleep
```


###远程管理

除了在本地控制systemd系统和服务管理器之外,systemctl实用程序还允许您通过ssh协议与在远程服务器上运行的systemd进行交互.需要使用openssh协议,并且远程服务器上openssh server开启

```
systemctl -H root@192.168.10.200 list-units --type service
root@192.168.10.200's password: 
  UNIT                                            LOAD   ACTIVE SUB     DESCRIPTION
  auditd.service                                  loaded active running Security Auditing Service
  chronyd.service                                 loaded active running NTP client/server
  crond.service                                   loaded active running Command Scheduler
  ...
```

###Systemd unit 配置文件指南

通常我们需要做一些自定义服务或者是许多编译软件并没有提供systemd的服务文件,就需要自己在/etc/systemd/system/目录中创建,编写unit文件.文件命名格式

```
unit_name.type_extension
unit_name:为服务名称
type_extension: 为unit类型包括,service,target,socket,device等等
```

同时unit文件允许为服务补充一个目录用于存放配置文件的符号链接,比如redis.service就可以在/etc/systemd/system下创建redis.service.d/limit.conf.类似的目录redis.service.wants和redis.service.requires

1, unit 文件结构
由三部分组成:

- [Unit]-unit的基本信息和参数,包括:服务说明,unit依赖于那些服务(man 5 systemd.unit)

```
Description		|	简要说明
Documentation	|	参考文档列表
After			|	当前服务启动之前必须要满足的条件(通常是target,service,socket),Before相反,定义当前服务之后的动作
Requires		|	其它依赖关系,要求列出的服务与当前服务一起激活,如果requires没启动成功,当前服务也失败
Wants			|	比Requires更弱的依赖关系,wants不会影响当前服务的激活.
Conflicts		|	互斥依赖和requires相反.
```

- [unit type]-根据unit类型特定的选项指令,比如service,target,socket,device(更多选项man 5 systemd.service, man 5 systemd.socket...)


```
Type			|	启动模式,配置影响execstart和服务进程相关选项
ExecStart		| 	服务启动所需要执行的命令或者脚本,还有ExecStartPre和ExecStartPost来指定ExecStar之前和之后的命令执行.Type=oneshot时可以指定多个,按顺序执行.
ExecStop		|	服务停止执行的命令或脚本
ExecReload		|	重新加载执行的命令或脚本
Restart			|	重启服务执行的命令或脚本
RemainAfterExit	|	默认值是false.如果设置为true即使所有进程退出,服务也被视为活动.Type=oneshot时特别有用
```

```
Type的模式
simple	|	默认值,以execstart开始的进程是服务的主要进程
forking	|	从execstart开始的进程产生一个子进程,成为服务的主进程.父进程在启动完成时退出
oneshot	|	和simple类似,但是在启动后续服务后退出进程
dbus	|	与simple类似,但在之前要获取一个d-bus name
notify	|	与simple类似,但是在之前要获取 sd_notify() 函数发送一个通知消息
idle	|	服务二进制的实际执行被延迟直到所有作业完成,这避免了状态输出与服务的shell输出的混淆
```

- [Install]-服务在enable和disable命令时所用的安装信息(man 5 systemd.unit)

```
Alias			|	为服务设置除systemctl enable外可使用的别名
RequiredBy		|	依赖列表,当服务被设为开机自启所需要的依赖列表
WantedBy		|	同上,弱依赖(通常设为WantedBy=multi-user.target)
Also			|	指定一起安装或卸载的设备列表
DefaultInstance	|	启用默认实例
```

###修改unit文件
大多数系统服务和rpm包的默认unit文件都存储在/usr/lib/systemd/system/,通过符号链接到/etc/systemd/system/目录,在需要对启动过程进程自定义修改是不建议直接修改源文件,官方建议是通过下面两种方法进行自定义

1, 创建配置文件目录/etc/systemd/system/unit.d/,在这个目录里做扩展配置和附加功能,具体步骤为:
> 创建目录
```
mkdir /etc/systemd/system/name.service.d/
```
> 创建扩展文件,必须为.conf
```
touch /etc/systemd/system/name.service.d/config_name.conf
```

> 修改功能,建议只创建专注于一个任务的小配置文件。

```
[Service]
ExecStartPost=/usr/local/bin/custom.sh
```

> 重新加载

```
systemctl daemon-reload
systemctl restart name.service
```
2, 在/etc/systemd/system/目录下创建单独的unit文件,而不是使用unit源文件的符号链接,具体步骤:

> 创建unit文件

```
cp /usr/lib/systemd/system/name.service /etc/systemd/system/name.service
```
> 修改
```
vi /etc/systemd/system/name.service
```
> 重新加载

```
systemctl daemon-reload
systemctl restart name.service
```


3, 观察unit文件的重写或修改的差异信息

```
systemd-delta
[EXTENDED]   /usr/lib/systemd/system/redis.service → /etc/systemd/system/redis.service.d/limit.conf
[REDIRECTED] /etc/systemd/system/default.target → /usr/lib/systemd/system/default.target
[EXTENDED]   /run/systemd/system/user-0.slice → /run/systemd/system/user-0.slice.d/50-Description.conf
...
```
关于差异的类型

```
[MASKED]		|	被mask的
[EQUIVALENT]	|	内容上没有区别的文件
[REDIRECTED]	|	重定向的文件(指向了其它文件)
[OVERRIDEN]		|	重写覆盖的文件
[EXTENDED]		|	使用/etc/systemd/system/unit.d/config_name.conf增加功能的文件
[UNCHANGED]		|	没有修改的文件
```


###My-test

1,启动一个redis的master-slave
> 复制.service文件
```
cd /etc/systemd/system/multi-user.target.wants
cp redis.service redis-m.service
cp redis.service redis-s1.service
cp redis.service redis-s2.service
```
> master
```
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/usr/bin/redis-server /opt/rs/7001/redis.conf --daemonize no
ExecStop=/usr/libexec/redis-shutdown
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```
> slave

```
[Unit]
Description=Redis persistent key-value database
After=network.target redis-m.service

[Service]
ExecStart=/usr/bin/redis-server /opt/rs/7002/redis.conf --daemonize no
ExecStop=/usr/libexec/redis-shutdown
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```

> 开启开机自启

```
#因为是放在默认的target目录下所以会自动启动,如果不是
systemctl enable redis-xx
```

2, 启动sentinel集群

> 复制.service文件

```
cd /etc/systemd/system
cp /usr/lib/systemd/system/redis-sentinel.service sentinel-1.service
```

> 配置文件

```
cat sentinel-1.service 
[Unit]
Description=Redis Sentinel
After=network.target redis-m.service

[Service]
ExecStart=/usr/bin/redis-sentinel /opt/sentinel/8001/redis-sentinel.conf --daemonize no
ExecStop=/usr/libexec/redis-shutdown redis-sentinel
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```
```
cat sentinel-2.service
[Unit]
Description=Redis Sentinel
After=network.target sentinel-1.service

[Service]
ExecStart=/usr/bin/redis-sentinel /opt/sentinel/8002/redis-sentinel.conf --daemonize no
ExecStop=/usr/libexec/redis-shutdown redis-sentinel
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```
```
cat sentinel-3.service
[Unit]
Description=Redis Sentinel
After=network.target sentinel-2.service

[Service]
ExecStart=/usr/bin/redis-sentinel /opt/sentinel/8003/redis-sentinel.conf --daemonize no
ExecStop=/usr/libexec/redis-shutdown redis-sentinel
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```

> 设置开机自启动

```
systemctl enable sentinel-1.service
```

需要注意的是因为systemd负责管理和创建服务进程.所以redis需要加上启动--daemonize no.



最后给几个例子

1, JMS和子项目coco

jms.service

```
[Unit]
Description=jms deamon
After=multi-user.target

[Service]
Type=simple
User=root
Group=root
Environment=VIRTUAL_ENV=/opt/py3/
Environment=PATH=/opt/py3/bin:$PATH
WorkingDirectory=/opt/jumpserver/
ExecStart=/opt/py3/bin/python /opt/jumpserver/run_server.py all
Restart=always

[Install]
WantedBy=multi-user.target
```

coco.service
```
[Unit]
Description=test deamon
After=network.target,jms.service

[Service]
Type=simple
User=root
Group=root
Environment=VIRTUAL_ENV=/opt/py3/
Environment=PATH=/opt/py3/bin:$PATH
WorkingDirectory=/opt/coco/
ExecStart=/opt/py3/bin/python /opt/coco/run_server.py start
ExecStop=/opt/py3/bin/python /opt/coco/run_server.py stop
Restart=always

[Install]
WantedBy=multi-user.target

```


2, 普通Django虚拟环境和uwsgi

```
[Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown www.www /run/uwsgi'
ExecStart=/data/py36/bin/uwsgi --emperor /etc/uwsgi/sites
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

