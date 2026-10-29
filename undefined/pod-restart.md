# pod이 restart 되어있을 때 이전 로그를 확인하는 방법

```
kubectl logs 에는 '--previous' 옵션이 있다. 
pod의 이름이 변경되지 않았다면 --previous 옵션을 사용하여 바로 직전의 로그 내용을 확인할 수 있다.

 > kubectl logs --previous {pod name}

```
