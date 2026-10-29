# AWS STG 노드 POD 수량 조절 작업

```
1. AWS STG EKS 노드 수량 조절 2 -> 4

2. POD 수량 조절
ks scale deploy integrateddomainmedicalsupport-deployment -n default --replicas=4
ks scale deploy integrateddomaincpoe-deployment -n default --replicas=10
ks scale deploy integrateddomainbilling-deployment -n default --replicas=4
ks scale deploy integrateddomainpharmacy-deployment -n default --replicas=6

ks scale deploy mq-integrateddomainmedicalsupport-deployment -n default --replicas=3
ks scale deploy mq-integrateddomaincpoe-deployment -n default --replicas=8
ks scale deploy mq-integrateddomainbilling-deployment -n default --replicas=4
ks scale deploy mq-integrateddomainpharmacy-deployment -n default --replicas=3

3. 작업 완료 후 POD 수량 조절

ks scale deploy integrateddomainmedicalsupport-deployment -n default --replicas=1
ks scale deploy integrateddomaincpoe-deployment -n default --replicas=1
ks scale deploy integrateddomainbilling-deployment -n default --replicas=1
ks scale deploy integrateddomainpharmacy-deployment -n default --replicas=1

ks scale deploy mq-integrateddomainmedicalsupport-deployment -n default --replicas=1
ks scale deploy mq-integrateddomaincpoe-deployment -n default --replicas=1
ks scale deploy mq-integrateddomainbilling-deployment -n default --replicas=1
ks scale deploy mq-integrateddomainpharmacy-deployment -n default --replicas=1

4. 축소 시작
기존 노드 보호 (i-01dd8306b7a8e07b7, i-04abe7512c4f00a1d)

EC2 -> Auto scaling -> 인스턴스 관리 -> 기존꺼 선택 후 축소보호설정

5. kubectl cordon {신규 노드 프라이빗 IP DNS 이름}
6. kubectl drain --ignore-daemonsets=true {신규 노드 프라이빗 IP DNS 이름}
7. 축소 4 -> 2
```
