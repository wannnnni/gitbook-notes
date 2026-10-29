# nginx 버전 업그레이드 (1.20.1 → 1.28.2)

## 🟢 1️⃣ 1.20.1 → 1.28.2 업그레이드 절차 (운영 기준)

### 🔹 사전 백업 (반드시)

```bash
nginx -T > /root/nginx_before_1.28_upgrade.conf
cp -r /etc/nginx /etc/nginx_backup_$(date +%Y%m%d)
rpm -qa | grep nginx > /root/pkg_before_upgrade.txt
```

***

### 🔹 1단계: nginx 중지

```bash
systemctl stop nginx
```

***

### 🔹 2단계: AppStream nginx 제거

```bash
yum remove nginx nginx-core nginx-filesystem nginx-mod-* -y
```

확인:

```bash
rpm -qa | grep nginx
```

→ 아무것도 없어야 정상

***

### 🔹 3단계: stable repo 설정

`/etc/yum.repos.d/nginx.repo`

```ini
[nginx-stable]
name=nginx stable repo
baseurl=https://nginx.org/packages/centos/9/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

```bash
yum clean all
yum makecache
```

***

### 🔹 4단계: 1.28.2 설치

```bash
yum install nginx-2:1.28.2-1.el9.ngx --nobest -y
```

확인:

```bash
nginx -v
```

***

### 🔹 5단계: 서비스 시작

```bash
systemctl start nginx
systemctl status nginx
```

다운타임: 보통 10\~30초

***

## 🔴 2️⃣ 롤백 방법

***

### ✅ 방법 A (nginx.org 내부 버전 롤백)

예: 1.28.2 → 1.25.3

```bash
systemctl stop nginx
yum install nginx-1:1.25.3-1.el9.ngx --nobest -y
systemctl start nginx
```

***

### ✅ 방법 B (완전 원복: 1.20.1 AppStream 복귀)

```bash
systemctl stop nginx
yum remove nginx -y
rm -f /etc/yum.repos.d/nginx.repo
yum install nginx -y
systemctl start nginx
```

버전 확인:

```bash
nginx -v
```

→ 1.20.1



## 🛡 3️⃣ CVE-2023-44487 대응 설정 (중요)

이 취약점은 HTTP/2 Rapid Reset 공격이야.

1.25 이상이면 기본 방어 로직 들어있지만\
**설정으로 추가 완화 권장**

***

### 🔹 nginx.conf → http {} 안에 추가

```nginx
keepalive_requests 1000;
http2_max_concurrent_streams 128;
```

***

### 🔹 추가 권장 (더 강력한 보호)

IP 기반 제한 추가 (권장)

```nginx
limit_req_zone $binary_remote_addr zone=req_per_ip:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=conn_per_ip:10m;

server {
    limit_conn conn_per_ip 50;
    limit_req zone=req_per_ip burst=20 nodelay;
}
```

***

### 🔹 적용

```bash
nginx -t
systemctl reload nginx
```
