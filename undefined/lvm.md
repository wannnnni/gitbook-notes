# LVM 구성

```
장치 확인 : fdisk -l

parted {장치명}
mklabel
gpt
mkpart primary 0% 100%
print    - 적용되었는지 확인
quit

pvcreate {장치명}1    - ex) /dev/sdb1
pvs

vgcreate data_vg /dev/sdb1
vgs

lvcreate -l 100%FREE -n data_lv data_vg
lvs

mkfs.xfs /dev/data_vg/data_lv

vi /etc/fstab    추가
systemctl daemon-reload
mount -a
```
