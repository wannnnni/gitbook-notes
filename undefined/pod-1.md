# 프로비저닝 반영 POD 재기동

```
1. 백엔드 활성화 작업

2. 인프라 tutorials 재기동 후 확인 요청
kubectl delete pod {tutorial-deployment-}

3. 백엔드 tutorials 확인
kubectl logs {tutorials-deployment-} <- 신규생성된 POD

4. 인프라 백엔드 일괄 재기동
kubectl rollout restart deployment -n default
```
