# env
系统环境配置,软件包安装 centos7   
```
yum update -y  
sed -i s/SELINUX=enforcing/SELINUX=disabled/g /etc/sysconfig/selinux  
# the file in some versions of the system are in /etc/selinux/config   
setenforce 0   
service iptables stop #or systemctl stop firewalld    
hostnamectl set-hostname cobbler     
echo "cobbler 192.168.10.250" >>/etc/hosts   
yum install -y chrony.x86_64 && systemctl start chronyd && systemctl enable chronyd   
timedatectl set-ntp 0   
timedatectl set-timezone Asia/Shanghai  
yum install bash-completion epel-release.noarch -y   
yum makecache  
reboot  
``` 

# service install 
服务安装,启动
```
yum -y install httpd dhcp tftp python-ctypes cobbler  xinetd cobbler-web   
systemctl start cobblerd  
systemctl enable cobblerd   
systemctl start httpd  
systemctl enable httpd   
systemctl start rsyncd   
systemctl enable rsyncd   
cobbler check  
```
```
[root@cobbler ~]# cobbler check
The following are potential configuration items that you may want to fix:

1 : The 'server' field in /etc/cobbler/settings must be set to something other than localhost, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it.
2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than 127.0.0.1, and should match the IP of the boot server on the PXE network.
3 : change 'disable' to 'no' in /etc/xinetd.d/tftp
4 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
5 : debmirror package is not installed, it will be required to manage debian deployments and repositories
6 : ksvalidator was not found, install pykickstart
7 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one
8 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes.
```
- 根据提示1.2,修改serverip为内网IP
```
sed -i 's/^server: 127.0.0.1/server: 192.168.10.250/' /etc/cobbler/settings   
sed -i 's/^next_server: 127.0.0.1/next_server: 192.168.10.250/' /etc/cobbler/settings  
```

- 修改3,tftp配置
```
vi /etc/xinetd.d/tftp  
disable			= no   
```
- 下载4,cobbler-loaders   
```
cobbler get-loaders  
```
- 安装6.8,pykickstart
```
yum install -y pykickstart fence-agents  
```
- 设置7,默认密码加密方式和默认密码

```
[root@cobbler ~]# openssl passwd -1 -salt "abcdef" "1234567890"
$1$abcdef$MgVBnksOqk99cgXuJlRJS0
reboot
```
此时重启服务器.让修改都生效第5项是deb的包管理工具.我用的centos7所以不管


#manage dhcp
- 启用dhcp功能
```
cat /etc/cobbler/settings|grep manage  
sed -i 's/manage_dhcp: 0/manage_dhcp: 1/g' /etc/cobbler/settings  
```
- 修改 dhcp 配置
vi /etc/cobbler/dhcp.template  
```
subnet 192.168.10.0 netmask 255.255.255.0 {
     option routers             192.168.10.1;
     option domain-name-servers 192.168.10.1;
     option subnet-mask         255.255.255.0;
     range dynamic-bootp        192.168.10.120 192.168.10.199;
```
```
systemctl restart cobblerd.service  
cobbler sync   
cat /etc/dhcp/dhcpd.conf #检查dhcp配置是否正确   
systemctl start dhcpd  
systemctl enable dhcpd   
```	 

#配置安装源(distro)	 
```
mount -t auto ./iso/CentOS-7-x86_64-Minimal-1511.iso  /mnt  
cobbler import --path=/mnt/ --name Centos7-x86_64 --arch=x86_64   
cobbler report #查看导入信息   
```

#自定义kickstart
vi centos7.cfg
```
###platform=x86, AMD64, or Intel EM64T
###version=DEVEL
### Install OS instead of upgrade
install

### Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'

### Root password
rootpw --iscrypted $default_password_crypted 
#Reference the cobbler password set earlier,see "openssl passwd"

### Use network installation
url --url="$tree"

### System language
lang en_US.UTF-8

### Firewall configuration
firewall --disabled

# System authorization information
auth  --useshadow  --passalgo=sha512

### Use text install
text
firstboot --disable

### SELinux configuration
selinux --disabled

### System services
services --enabled="chronyd"

### Network information
network  --bootproto=dhcp --device=eth0  
#network  --bootproto=static --device=eth0 --gateway=192.168.10.1 --ip=192.168.10.188 --nameserver=114.114.114.114 --netmask=255.255.255.0 --ipv6=auto --activate
#network  --hostname=test-ks

### Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai --isUtc --ntpservers=0.centos.pool.ntp.org,1.centos.pool.ntp.org,2.centos.pool.ntp.org,3.centos.pool.ntp.org

### System bootloader configuration
bootloader --location=mbr
autopart --type=lvm

### Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --asprimary --fstype="ext4" --size=200
part swap --fstype="swap" --size=1024
part / --fstype="ext4" --grow --size=1

%pre
$SNIPPET('log_ks_pre')
$SNIPPET('kickstart_start')
$SNIPPET('pre_install_network_config')
# Enable installation monitoring
$SNIPPET('pre_anamon')
%end

%packages
@^minimal
@core
chrony
%end

%addon com_redhat_kdump --disable --reserve-mb='auto'

%end	

%post

%end 

```
```
cp centos7.cfg /var/lib/cobbler/kickstarts/   
cobbler profile edit --name Centos7-x86_64 --kickstart=/var/lib/cobbler/kickstarts/centos7.cfg  
cobbler profile edit --name Centos7-x86_64 --kopts='net.ifnames=0 biosdevname=0'     #修改默认网卡名为eth0  
cobbler validateks #检查ks文件配置  
cobbler sync  
```
#定制装机页面
```
vi /etc/cobbler/pxedefault.template 
MENU TITLE who-what-when Install System   #修改title
MENU MASTER PASSWD $1$abcdef$MgVBnksOqk99cgXuJlRJS0  #pxe passwd
LABEL XXXX
	MENU PASSWD   #启用密码
```

#新建虚拟机使用pxe启动	 
```
qemu-img create -f qcow2 -o size=60G /var/lib/libvirt/images/test.qcow2   
virt-install --name test --ram 1024 --vcpus=2 --pxe --disk path=/var/lib/libvirt/images/  test.qcow2,bus=virtio,size=20,sparse,format=qcow2 --network bridge=br0,model=virtio --force	   
```
#客户端自动重装 
```
yum install -y epel-release  
yum install -y koan  
koan --server=192.168.10.250 --list=profiles  
koan --replace-self --server=192.168.10.250 --profile=Centos7-x86_64   
```	 
#distro,profile管理
```
cobbler distro copy --name xxx --newname yyy   
cobbler profile copy --name xxx --newname yyy   
```
>  其它add copy edit 同理.看看help

#kickstart配置
```
yum install system-config-kickstart -y   
system-config-kickstart	 #安装x11可以图形化设置生成ks文件  
```	 
#cobbler-web	
```
htdigest /etc/cobbler/users.digest "Cobbler" cobbler #改密码  
htdigest /etc/cobbler/users.digest "Cobbler" sylar #新建用户  
```
https://192.168.10.250/cobbler_web  #注意必须使用https   

#cobbler-dhcp冲突
如果在一个局域网中有多个dhcp服务器的，开启cobbler上的dhcp服务会干扰目前正在运行的dhcp服务器的话，只需把cobbler上的dhcp的range网络段给注释掉即可，就是变成只能指定分配ip.同时使用不同的网段
```
cobbler system add --name=network --ip=172.16.0.0/16 --profile=net-x86_64

```



