# AWS STG-NCP STG 대상 VPN 작업

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>NCP -> Server -> Network Interface 에서 비공인 IP 추가</p></figcaption></figure>



해당 서버 접속 후&#x20;

```
cd /etc/sysconfig/network-scripts
vi ifcfg-eth1
```

<div align="left"><figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>ip 설정</p></figcaption></figure></div>

```
ifup eth1
ip a -> ip 잘 들어갔는지 확인
route 설정
route add -net 172.26.0.0 netmask 255.255.0.0 gw 192.168.99.1 dev eth1
```

이후 서버 간 통신 확인인
