# SSL 인증서 갱신

```
  콘솔 -> AWS Certificate Manager -> 기존인증서 선택 -> 다시가져오기

1.	PEM 파일 안 [인증서 본문] 내용 붙여넣기
-	_wildcard_.edgennext.com_202208226119F.crt.pem
2.	PEM 파일 안 [인증서 프라이빗 키] 내용 붙여넣기
-	_wildcard_.edgennext.com_202208226119F.key.pem
3.	PEM 파일 안 [인증서 체인] 내용 붙여넣기
-	ca-chain-bundle.pem


 ※ 주의사항 : 
1. 해당 파일안의 텍스트를 전부 복사하여 붙여넣기 하는데 처음과 끝 부분 공백이 생기지 않도록 붙여넣는다. 
   (-----BEGIN CERTIFICATE----- xxxxxxxxxxxxxxxxxxxx -----END CERTIFICATE-----)
2.  AWS는 서비스 다운타임없이 전환되지만 사용자의 통신 상태에 따라서 순단 발생 가능

 Load Balance
갱신시 LB는 교체 안해도됨.
다만, 인증서 교체일 때는 LB에서도 교체해줘야함.

```
