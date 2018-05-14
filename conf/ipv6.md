### Centos7
```
#vi /etc/default/grub
GRUB_CMDLINE_LINUX="ipv6.disable=1 rd.lvm.lv=centos_cobbler/root rd.lvm.lv=centos_cobbler/swap rhgb quiet"

```
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```