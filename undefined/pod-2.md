# POD 수량조절

```
[pod 수량조절 - 0으로 만들면 off]
kubectl scale deploy {deployment} --replicas=0
ex) kubectl scale deploy integrateddomaintechnicalservice-deployment --replicas=0

[pod 수량조절 - 1로 만들면 on]
kubectl scale deploy {deployment} --replicas=1
ex) kubectl scale deploy integrateddomaintechnicalservice-deployment --replicas=1



************POD 각각 재기동************
kubectl delete pod {pod-name}

deploy전체 : kubectl rollout restart deploy {deployment} -n default

전체 재기동 **************주의****************
kubectl rollout restart deployment -n default

운영 개발기 전체 재기동 **************주의****************
kubectl rollout restart deployment
```
