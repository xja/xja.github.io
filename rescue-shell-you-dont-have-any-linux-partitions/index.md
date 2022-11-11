# Rescue Shell - You Don't Have Any Linux Partitions


系统本身是正常的，在测试 rescue 环境的时候发现系统分区不能自动挂载

```BashSession
Rescue Shell

You don't have any Linux partitions.
When finished, please exit from the shell and your system will reboot.
Please press ENTER to get a shell:
sh-4.4# fdisk -l
Disk /dev/sda: 20GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optiomal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd23696aa

Device     Boot    Start       End   Sectors  Size  Id  Type
/dev/sda1  *        2048   2099199   2097152    1G  83  Linux
/dev/sda2        2099200  41943039  39843840   19G  8e  Linux LVM
sh-4.4# mount /dev/sda2 /mnt/sysimage
mount: /mnt/sysimage: unknown filesystem type 'LVM2_member'.
sh-4.4# vgchange -ay
  2 logical volumes(s) in volume group "cs" now active
sh-4.4# lvscan
  ACTIVE             '/dev/cs/swap' [2.00 GiG] inherit
  ACTIVE             '/dev/cs/root' [<17.00 GiG] inherit
sh-4.4# mount /dev/cs/root /mnt/sysimage
sh-4.4# chroot /mnt/sysimage
bash-4.4# cat /etc/*-release
CentOS Stream release 8
NAME="CentOS Stream"
VERSION="8"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Stream 8"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_SUPPORT_PRODUCT_VERSION="CentOS Stream"
CentOS Stream release 8
CentOS Stream release 8
bash-4.4# 
```

## Reference

1. [14.04 - mount unknown filesystem type 'lvm2_member' - Ask Ubuntu](https://askubuntu.com/a/1078061)

