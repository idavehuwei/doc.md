### 学习工具

1,废旧电脑一台或虚拟机(vbox或vm自选),虚拟机 [下载地址](https://www.virtualbox.org/wiki/Downloads)

2,putty(xshell): 终端工具,[下载地址](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) putty-connection-manager [下载之地](http://putty-connection-manager.software.informer.com/download/)

3,xming(xmanager):X-window服务 [网站链接](https://xming.en.softonic.com/)

4,rufus:u盘制作工具 [下载地址](http://rufus.akeo.ie/downloads/rufus-2.18.exe)

5,centos:系统镜像 [下载地址](https://mirrors.aliyun.com/centos/7.4.1708/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso)

6,各类网站,资源镜像站点,官方文档教程,搜索站点(google,baidu)



###centos7文本安装过程
1,文本安装选择(默认为图形): 安装选择界面--->esc --->boot提示符输入: linux text

2,进入文本安装界面有9个选项,分别是:
```
1	| 	语言选择
2	|	时区选择
3	|	安装源(一般不用设置使用默认介质)
4	|	安装包组(使用的mini,所以只有base包组,一般也不用设置)
5	|	安装介质
6	|	Kdump	(可配可不配置)
7	|	网络设置
8	|	root密码设置
9	|	用户设置
另外分别其他几个不同步骤下的热键
q	|	退出
r	|	刷新
c	|	继续
b	|	返回或开始安装
```

3, 选择语言:主选项输入: 1 回车 ---> 找到english(16)或Chinese代码输入: 68 回车 --->进入语种选择大陆简体代码输入:1 回车 

4, 选择时区:主选项输入: 2 回车 ---> 找到亚洲asia代码输入: 5 回车 ---> 找到上海代码输入: 62 回车

5, 配置硬盘分区: 主选项输入: 5 回车 ---> 进入硬盘选择一般不用选择输入:c 回车 ---> 进入硬盘空间根据需要输入(1 重分配存在的分区,2使用所有空间,3使用空余空间):2 回车
 ---> 再次确认,输入: c 回车 ---> 选择分区格式输入: 3 回车

6, 配置网络:	主选项输入: 7 回车  ---> 选择配置网卡,输入: 2 回车 ---> 配置ip,输入: 1 回车 ---> 192.168.10.66 回车 ---> 配置掩码,输入: 2 回车 ---> 255.255.255.0 ---> 配置网关,输入:3 回车 ---> 192.168.10.1 回车 ---> 配置dns,输入: 6 回车 ---> 114.114.114.114 回车  ----> 7 回车 ---> 8 回车 ---> 保存配置回主选项输入两次:c 回车

7, 配置root密码: 主选项输入: 8 回车 ---> 输入两次密码回车 ---> 确认密码:yes 回车 

8, 开始安装:主选项输入: b 回车 

> 在文本模式选择的时候,大部分选项选中之后又一个[X]标记,   
> 同一个选项选择两次会取消选择!
> root密码一定要设置不然进入不了系统

###修复root 密码

1,重启到内核选择界面输入:e

2,找到有LANG=zh_CN.UTF-8的行,在行尾添加: init=/bin/bash

3,将这行前面的一处ro修改为:rw

4,CTRL+x启动系统

5,使用passwd修改密码

6,由于selinux的保护并不能真正修改,这时输入:touch /.autorelabel

7,重启:exec /sbin/init


















