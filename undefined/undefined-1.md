# 스토리지 생성 후 마운트

```
#!/bin/bash
mkdir /box
(echo p; echo n; echo p; echo 1;  echo ; echo ; echo t; echo 8e; echo w) | fdisk /dev/xvdb
partprobe
pvcreate /dev/xvdb1
vgcreate box /dev/xvdb1
lvcreate  -l +100%FREE -n box box
mkfs -t xfs /dev/mapper/box-box
echo '/dev/mapper/box-box     /box                   xfs     defaults        0 0' >> /etc/fstab
mount -a
```
