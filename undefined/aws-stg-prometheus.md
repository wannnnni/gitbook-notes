# AWS STG prometheus 스토리지 증설

```
1. prometheus의 PVC 찾기
   kubectl get pvc -n <namespace>

2. PV의 용량 확인
   kubectl describe pv <pv-name>

3. AWS에서 해당 볼륨 증설

4. PVC 크기 확장(PVC를 편집하여 spec.resources.requests.storage를 원하는 크기로 업데이트)
   kubectl edit pvc <pvc-name> -n <namespace>

5. 크기 변경 확인
   kubectl get pvc <pvc-name> -n <namespace>
   kubectl get pv -o wide

```
