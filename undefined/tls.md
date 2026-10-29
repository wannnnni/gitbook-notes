# TLS 점검 방법

TLS 점검 방법

```
openssl s_client -connect stage-spellchecker.edgennext.com:443 -tls1 
openssl s_client -connect stage-spellchecker.edgennext.com:443 -tls1_1
openssl s_client -connect stage-spellchecker.edgennext.com:443 -tls1_2
openssl s_client -connect stage-spellchecker.edgennext.com:443 -tls1_3
```

