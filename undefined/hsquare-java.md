# hsquare java 모니터링 구축

```
1. hsquare deploy가 mount한 pv 실제 경로 찾기

2. 해당 경로에서 
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.16.1/jmx_prometheus_javaagent-0.16.1.jar

vi jmx_exporter.yaml
rules:
  - pattern: '.*'
wq:

3. 서비스 등록
kubuctl edit service hsquare-service -n prometheus -o yaml

  - name: jmx
    nodePort: 30887
    port: 8087
    protocol: TCP
    targetPort: 8087

4. configmap 수정
kubectl edit configmap prometheus-server -n prometheus -o yaml

5. deploy 수정
kubectl edit deploy hsquare-deployment -n default -o yaml

     - env: 
        - name: JAVA_TOOL_OPTIONS
          value: -javaagent:/box/jmx_prometheus_javaagent-0.16.1.jar=8087:/box/jmx_exporter.yaml

6. hsquare 재기동
kubectl delete pod {pod name} -n default

7. prometheus-server 재기동
kubectl delete pod {pod name} -n prometheus

8. 그라파나 data source 생성
```
