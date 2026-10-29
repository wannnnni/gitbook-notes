# JD pod 구축 정보

JD POD 구축 정보

이미지를 불러와야하기에 secret 를 구성해야한다

JD 환경정보

```
[Prod]
- name: MEDICALERP_PROJECT
  value: ezcaretech
- name: MEDICALERP_SERVER
  value: Production
- name: MEDICALERP_DOMAIN
  value: edgennext.com
image : medicalerp.kr.ncr.ntruss.com/medicalerp:latest

[Stage]
- name: MEDICALERP_PROJECT
  value: ezcaretech
- name: MEDICALERP_SERVER
  value: Staging
- name: MEDICALERP_DOMAIN
  value: edgennext.com
image : medicalerp-stg.kr.ncr.ntruss.com/medicalerp-stg:latest
```

다른 환경의 dockerconfigjson을 긁어서 [https://www.base64decode.org/](https://www.base64decode.org/) DECODE 진행

거기 나오는 계정값으로 secret 생성

```
kubectl create secret docker-registry ezpc-medicalerp \
  --docker-server=medicalerp.kr.ncr.ntruss.com \
  --docker-username=DF7FB828BEAA6EB1103E \
  --docker-password=A0C9ED9A90E017F48D0544AD68A3909DEF93BC51 \
  --docker-email=baesungmin@ezcaretech.com
```

이후 deployment 수정

```
containers
- env 값 수정
imagePullSecrets 값 secret 이름으로 변경
```
