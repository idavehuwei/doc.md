# 利用废旧机器搭建kvm虚拟化平台

### 需要工具

1,废旧电脑一台

2,putty(xshell): 终端工具,[下载地址](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

3,xming(xmanager):X-window服务 [网站链接](https://xming.en.softonic.com/)

4,rufus:u盘制作工具 [下载地址](http://rufus.akeo.ie/downloads/rufus-2.18.exe)

5,centos:系统镜像 [下载地址](https://mirrors.aliyun.com/centos/7.4.1708/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso)


### 安装centos7
下一个教程中介绍

### yum安装kvm
```
yum -y install qemu-kvm python-virtinst libvirt tunctl bridge-utils  virt-manager qemu-kvm-tools virt-viewer virt-v2v virt-install tigervnc-server dejavu-lgc-sans-fonts
virsh iface-bridge eth0 br0  #设置桥接网络
```


### console
```
grubby --update-kernel=ALL --args="console=ttyS0"
reboot
...
virt-install xxxxxxxxxxxxx --extra-args='console=ttyS0'
```


### clone script

```
#!/bin/bash
echo -e  "exec this script:  \033[32;40;5mbash vm-clone.sh from_vm_name new_vm_name\033[0m"

from_model=$1
new_vm=$2

virt-clone -o $from_model -n $new_vm --file=/var/lib/libvirt/images/vm/$new_vm.qcow2

```





