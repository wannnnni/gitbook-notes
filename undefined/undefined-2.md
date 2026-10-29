# 스토리지 삭제

```
lvm 삭제
sudo lvremove /dev/mapper/box-box

vg 삭제
sudo vgremove /dev/mapper/box

pv 삭제
lsblk
sudo pvremove /dev/{ }

/etc/fstab에서 삭제

AWS 볼륨 분리 -> 삭제
```
