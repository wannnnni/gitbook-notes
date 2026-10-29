# 특정POD의 registry값 확인

```

방법 1: kubectl 명령어 사용
1. Pod의 이름과 네임스페이스 확인
먼저, 해당 Pod의 이름과 네임스페이스를 확인해야 합니다. 
예를 들어, default 네임스페이스에 있는 my-pod라는 이름의 Pod를 확인한다고 가정하겠습니다.

2. Pod의 YAML 출력
kubectl get pod 명령어를 사용하여 해당 Pod의 상세 정보를 YAML 형식으로 출력합니다.

kubectl get pod my-pod -n default -o yaml

3. 이미지 경로 확인
출력된 YAML 파일에서 containers 섹션 아래의 image 필드를 확인합니다. 
이 필드에 컨테이너 이미지의 레지스트리 경로가 포함되어 있습니다. 예를 들어:

containers:
- name: my-container
  image: myregistry.com/myimage:tag
여기서 myregistry.com/myimage:tag가 이미지의 레지스트리 경로입니다.
```
