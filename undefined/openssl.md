---
description: p7b 인증서 jks 로 변환 방법
---

# openssl 인증서 변환

{% code overflow="wrap" fullWidth="false" %}
```
[p7b -> jks변환 - 체인 비포함] 
openssl pkcs7 -print_certs -in remote.neoemr.mohw.go.kr.p7b -out remote.neoemr.mohw.go.kr.jks.cer\n keytool -importcert -file remote.neoemr.mohw.go.kr.jks.cer -keystore remote.neoemr.mohw.go.kr.jks -alias remote.neoemr.mohw.go.kr
keytool -list -keystore remote.neoemr.mohw.go.kr.jks
```
{% endcode %}

{% code overflow="wrap" %}
```
[p7b -> jks변환 - 체인 포함] 
openssl pkcs12 -export -name example.com -in in.pem -inkey key_d.pem -out out.pfx
keytool -importkeystore -srckeystore out.pfx -srcstoretype pkcs12 -destkeystore out.jks -deststoretype jks
```
{% endcode %}



