
### firewall工作原理 

firewall分为两层D-BUS暴露给外界的命令行工具接口,Core核心工作层.firewall-offline-cmd可以绕过D-BUS层取直接修改防火墙规则,但是不推荐使用.   

![](https://www.firewalld.org/documentation/firewalld-structure+nftables.png)


firewalld推荐运行于NetworkManager软件环境.


### iptables
ubuntu 上强制使用iptables

##### iptables配置
```
ip6tables -P INPUT DROP
ip6tables -P OUTPUT DROP
ip6tables-save > /etc/ip6tables.rules
echo -e "ip6tables-restore </etc/ip6tables.rules" >/etc/network/if-pre-up.d/ip6tables
chmod +x /etc/network/if-pre-up.d/ip6tables
```
##### 基础配置
```
iptables -A INPUT -i lo -j ACCEPT 
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
iptables -A INPUT -p tcp  --dport 22022 -j ACCEPT
...
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT 
iptables -P INPUT DROP
```


### ufw

##### 首先关闭IPV6
```
#/etc/default/ufw
IPV6=no
```

##### 服务开关
```
ufw enable 	#默认会将INPUT链状态改为DROP
ufw disable
```
##### 状态查看
```
ufw status				|	服务状态查看
ufw status verbose		|  	查看规则信息
ufw status numbered 	|	查看规则编号
ufw show raw 			|	查看原始规则,/etc/ufw/下
```

##### 基本策略

```
通过			|		ufw allow 22022/tcp	
阻止			|		ufw deny 22/tcp
删除			|		ufw delete 22/tcp	#也可以使用numbered查看的规则编号
插入			|		ufw insert 1 allow 222
```

















