# 해외시연기 EKS 업그레이드 이슈

```
기존 node 삭제 후 신규 node 설정 하였을때 헬스체크 unhealthy

aws-os-webproxy-001 /etc/nginx/nginx.conf 에서 upstream frontend에 있는
server [node IP:30800] max_fails=0;
위 코드를 신규 node IP로 변경해야함.


신규 노드 보안그룹에 0.0.0.0/0 모든트래픽으로 설정되어있어야함.



```
