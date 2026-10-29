# /root 볼륨 증설

### 1. Hyper-V에서 VHDX 확장

마스터 이미지 VM은 가능하면 종료한 상태에서 진행합니다.

1. Hyper-V 관리자 실행
2. 대상 VM 우클릭 → **설정**
3. root 디스크가 연결된 하드 드라이브 선택
4. **편집**
5. `확장` 선택
6. 크기를 `200GB`로 입력
7. 완료

### 2. VM 부팅 후 디스크 크기 확인

VM을 시작한 뒤 다음 명령을 실행합니다.

```
lsblk
```

예상 결과:

```
sda           200G
├─sda1        600M
├─sda2          1G
└─sda3       48.4G
```

여기서 `/dev/sda`만 200GB이고 `/dev/sda3`는 기존 크기여도 정상입니다.

`parted`에서는 200GB가 약 215GB로 표시될 수 있습니다. 이는 단위 차이입니다.

```
200 GiB ≒ 215 GB
```

### 3. GPT 파티션 테이블 수정

Hyper-V에서 디스크가 커졌지만 GPT의 백업 헤더는 기존 50GB 위치에 남아 있을 수 있습니다.

다음 명령을 실행합니다.

```
parted /dev/sda
```

`parted` 내부에서:

```
print
```

다음 경고가 나오면:

```
Warning: Not all of the space available to /dev/sda appears to be usedFix/Ignore?
```

반드시 다음과 같이 입력합니다.

```
Fix
```

그다음 다시 확인합니다.

```
print
```

### 4. sda3 파티션 확장

같은 `parted` 화면에서 실행합니다.

```
resizepart 3 100%
```

사용 중인 파티션이라는 경고가 나오면:

```
Yes
```

를 입력합니다.

완료 후 종료합니다.

```
quit
```

전체 입력 흐름은 다음과 같습니다.

```
[root@server ~]# parted /dev/sda
(parted) printFix/Ignore? Fix
(parted) resizepart 3 100%
Warning: Partition /dev/sda3 is being used.Continue? Yes
(parted) quit
```

### 5. 커널에 파티션 변경 사항 반영

```
partprobe /dev/sdaudevadm settlelsblk
```

정상적으로 반영되면 `/dev/sda3`가 약 198GB 수준으로 표시됩니다.

```
sda          200G
├─sda1       600M
├─sda2         1G
└─sda3     198.4G
```

여전히 `/dev/sda3`가 48.4GB라면 서버를 재부팅합니다.

```
reboot
```

재부팅 후 확인합니다.

```
lsblk
```

### 6. LVM Physical Volume 확장

파티션은 커졌지만 LVM은 아직 기존 용량만 사용합니다.

```
pvresize /dev/sda3
```

확인:

```
pvsvgs
```

예상 결과:

```
PV         VG  PSize    PFree/dev/sda3  rl  <198G    <150G
```

`PFree` 또는 `VFree`에 약 150GB가 표시되면 정상입니다.

### 7. root Logical Volume 확장

VG의 남은 공간을 `/`에 모두 할당합니다.

```
lvextend -l +100%FREE /dev/mapper/rl-root
```

확인:

```
lvs
```

### 8. XFS 파일시스템 확장

파일시스템 종류를 먼저 확인합니다.

```
df -Th /
```

`xfs`라면:

```
xfs_growfs /
```

`ext4`라면:

```
resize2fs /dev/mapper/rl-root
```

현재 Rocky Linux 계열 기본 구성이라면 일반적으로 XFS입니다.

### 9. 최종 확인

```
df -Th /lsblkpvsvgslvs
```

예상 결과:

```
Filesystem           Type  Size  Used Avail Use% Mounted on/dev/mapper/rl-root  xfs   194G  ...  ...   ...  /
```

디스크 전체가 200GB여도 `/`는 EFI, `/boot`, swap 공간을 제외하므로 약 194GB 정도로 표시됩니다.

### 전체 명령 요약

Hyper-V에서 VHDX를 200GB로 확장한 뒤:

```
lsblkparted /dev/sda
```

`parted` 내부:

```
printFixresizepart 3 100%Yesquit
```

이후:

```
partprobe /dev/sdaudevadm settlelsblkpvresize /dev/sda3pvsvgslvextend -l +100%FREE /dev/mapper/rl-rootxfs_growfs /df -Th /lsblk
```

`lvextend`와 파일시스템 확장을 한 번에 처리하려면 다음 명령도 사용할 수 있습니다.

```
lvextend -r -l +100%FREE /dev/mapper/rl-root
```

이 경우 `xfs_growfs /`는 별도로 실행하지 않아도 됩니다.
