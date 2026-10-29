# NAS 마운트

```
#mount -t nfs [NAS IP주소]:[공유 폴더 경로] [마운트할 경로]

# fstab 등록
[NAS IP주소]:[공유 폴더 경로] [마운트할 경로] nfs defaults 0 0

ubuntu의 경우 apt-get install -y nfs-common cifs-utils 설치
```
