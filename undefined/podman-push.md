# podman push

gitlab에 이미지 push

```
podman pull grafana/grafana:12.0.2
podman tag docker.io/grafana/grafana:12.0.2 gitlab.com.ezpc.internal:5050/root/monitoring/grafana:12.0.2
podman push gitlab.com.ezpc.internal:5050/root/monitoring/grafana:12.0.2
```

