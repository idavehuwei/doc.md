

### LVM创建

```
fdisk /dev/sdb
n
p
...

t
8e
w
#格式化硬盘

mkdir /vm-data								#创建目录
pvcreate /dev/sdb3							#pvcreate $disk  							分区加入PV
vgcreate vm-data /dev/sdb3					#vgcreate $vgname $disk  					设置VG卷标
lvcreate -l 100%VG -n lv-vm-data vm-data	#lvcreate -l 100%VG -n $lvmname $vgname  	划分LV容量
mkfs.ext4 /dev/vm-data/lv-vm-data			#mkfs.ext4 /dev/$vgname/$lvmname			格式化
echo "/dev/vm-data/lv-vm-data /vm-data        ext4    defaults        0 0" >> /etc/fstab			#挂载
mount -a 									#测试挂在是否报错
```

### LVM vg扩容(新增硬盘的方式)

##### xfs格式
```
pvcreate /dev/sdb1
vgextend vm-data /dev/sdb1 
lvextend -L +800G /dev/vm-data/lv-vm-data 
xfs_growfs /dev/mapper/centos-root
```
##### ext4格式
```
pvcreate /dev/sdb1
vgextend vm-data /dev/sdb1 
lvextend -L +800G /dev/vm-data/lv-vm-data
resize2fs /dev/mapper/vm--data-lv--vm--data
```

### LVM lv缩减(和lv扩容结合使用,在不增加硬盘时合理重分配LVM分区大小)

##### ext4格式，可动态缩减

```
df -h
fuser -m /home
umount /home
fsck -f /dev/mapper/centos00-home
resize2fs /dev/mapper/centos00-home 10G
lvresize --size 10G /dev/mapper/centos00-home
```
##### xfs格式,不支持动态缩减。必须重新格式化

```
df -h
fuser -m /home
umount /home
lvreduce -L -24G /dev/mapper/centos00-home #减少xfs大小
mkfs.xfs /dev/mapper/centos00-home -f      #格式化
```

### LVM lv扩容

##### ext4
```
lvextend -L +574G /dev/centos00/root
resize2fs /dev/mapper/centos00-root
```

##### xfs格式
```
lvextend -L +574G /dev/centos00/root
xfs_growfs /dev/mapper/centos00-root
df -hT
```

