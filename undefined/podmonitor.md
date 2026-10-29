# podmonitor 설정

등록방법

강제구성파일 "EnableAppMetrics": true 확인

\##하위버전##

1. service에 속성 추가해야함(api, mq 모두) 개발 서버 mq-tutorials, tutorials 서비스 참고
2. prometheus configmap 수정

kubectl edit cm prometheus-server -n monitoring

*   job\_name: kubernetes-service-endpoints kubernetes\_sd\_configs: - role: endpoints relabel\_configs: # ... 기존에 있던 리라벨링 설정들 ...

    ```
    # [★ 맨 아래에 이 3줄 추가]
    - source_labels: [__meta_kubernetes_pod_name]
      action: replace
      target_label: pod
    ```

kubectl rollout restart deployment/prometheus-server -n monitoring

or

lock 걸리면서 제대로 재시작 되지 않는 경우

## 1) 스케일 다운 (프로메테우스 완전 중지)

kubectl scale deployment prometheus-server -n monitoring --replicas=0

## 2) 파드가 완전히 사라졌는지 확인

kubectl get pods -n monitoring | grep prometheus-server

## 3) 다시 스케일 업 (프로메테우스 재시작)

kubectl scale deployment prometheus-server -n monitoring --replicas=1

\##상위버전##

podmonitors 추가(api, mq 모두)

cat <\<EOF | kubectl apply -f - apiVersion: monitoring.coreos.com/v1 kind: ServiceMonitor metadata: name: tutorials namespace: default spec: endpoints: - targetPort: 16099 # 서비스 포트 path: /metrics selector: matchLabels: app: tutorials # tutorials-service 서비스의 라벨 EOF

dashboard 메트릭 참고 http://10.1.40.33:3000/d/OnHKphr7z/kubernetes-monitoring-v0-1?editPanel=355\&orgId=1\&refresh=15s
