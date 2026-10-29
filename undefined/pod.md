# 쿠버네티스 POD 조회 권한

var config = KubernetesClientConfiguration.InClusterConfig();\
\=> 기본적으로 kubernetes default 서비스 계정을 사용함\
sa(service account) default에 pod read 권한 줘야함

<pre><code><strong>kubectl apply -f - &#x3C;&#x3C;EOF
</strong>apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: api
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default-pod-reader
  namespace: api
subjects:
- kind: ServiceAccount
  name: default
  namespace: api
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EOF
</code></pre>
