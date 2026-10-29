# Opensearch 버전 업그레이드

**업그레이드 순서: stage-logstackdb-01 → stage-logstackdb-03 → stage-logstackdb-02** (현재 매니저인 stage-logstackdb-02이 마지막)

***

#### A. 시작 전 확인

먼저 비밀번호를 히스토리에 남기지 않도록 변수로 잡아 두세요.

bash

```bash
read -s PW
export OS="https://localhost:19200"
```

**복제본 0인 인덱스 확인.** `rep` 값이 0인 인덱스가 있으면 해당 노드 정지 중에 클러스터가 yellow/red가 되고, green 대기가 영원히 끝나지 않습니다.

bash

```bash
curl -sk -u admin:$PW "$OS/_cat/indices?v&h=health,index,pri,rep,docs.count,store.size" | sort -k4
```

0인 인덱스가 있다면 두 가지 선택지가 있습니다. 업그레이드 기간 동안만 복제본을 1로 올리거나(권장), 또는 green 대신 yellow를 기준으로 진행하는 것입니다.

bash

```bash
# 임시로 복제본 1 부여
curl -sk -u admin:$PW -X PUT "$OS/문제인덱스/_settings" \
  -H 'Content-Type: application/json' -d'{"index":{"number_of_replicas":1}}'
```

**디스크 여유 확인.** 노드 하나가 빠진 동안 나머지 2대로 샤드가 몰릴 수 있습니다.

bash

```bash
curl -sk -u admin:$PW "$OS/_cat/allocation?v"
df -h /var/lib/opensearch
```

**복제본 재구성 지연 시간 상향.** 노드 재시작 몇 분 동안 불필요한 샤드 복제가 일어나지 않게 합니다. 업그레이드 내내 유지했다가 끝나고 되돌리면 됩니다.

bash

```bash
curl -sk -u admin:$PW -X PUT "$OS/_all/_settings" \
  -H 'Content-Type: application/json' -d'{"settings":{"index.unassigned.node_left.delayed_timeout":"10m"}}'
```

***

#### B. 백업 (3대 모두에서)

bash

```bash
# 각 노드에서
sudo tar czf /root/os-config-3.1.0-$(date +%F).tar.gz /etc/opensearch
sudo /usr/share/opensearch/bin/opensearch-plugin list | tee /root/plugins-3.1.0.txt
```

스냅샷 리포지터리가 설정되어 있는지 확인하고, 있다면 찍어 두세요.

bash

```bash
curl -sk -u admin:$PW "$OS/_snapshot?pretty"
curl -sk -u admin:$PW -X PUT "$OS/_snapshot/<리포지터리명>/pre-330-$(date +%Y%m%d)?wait_for_completion=true"
```

***

#### C. 파일 반입

외부망에서 받아서 세 노드 모두에 뿌립니다.

bash

```bash
# 외부망 PC
BASE=https://artifacts.opensearch.org/releases/bundle/opensearch/3.3.0
curl -O $BASE/opensearch-3.3.1-linux-x64.rpm
sha512sum opensearch-3.3.1-linux-x64.rpm > checksum.txt

# 경유 서버 → 각 노드
for ip in 172.30.113.38 172.30.113.39 172.30.113.40; do
  scp opensearch-3.3.0-linux-x64.rpm checksum.txt root@$ip:/tmp/
done
```

**세 노드 각각에서 검증합니다.**

bash

```bash
cd /tmp && sha512sum -c checksum.txt
```

***

#### D. 1번째: stage-logstackdb-01 (172.30.113.38)

**D-1. 할당 비활성화 + flush** (아무 노드에서나 1회)

bash

```bash
curl -sk -u admin:$PW -X PUT "$OS/_cluster/settings" \
  -H 'Content-Type: application/json' -d'
{ "persistent": { "cluster.routing.allocation.enable": "primaries" } }'

curl -sk -u admin:$PW -X POST "$OS/_flush"
```

**D-2. stage-logstackdb-01에서 중지 및 설치**

bash

```bash
# ssh root@172.30.113.38
sudo systemctl stop opensearch
sudo rpm -Uvh /tmp/opensearch-3.3.1-linux-x64.rpm
```

**D-3. 설정 파일 점검**

bash

```bash
sudo find /etc/opensearch -name "*.rpmnew" -o -name "*.rpmsave"
```

`.rpmsave`가 나오면 내 설정이 밀려난 것이므로 복원합니다. 특히 `opensearch.yml`에 포트 19200 설정과 클러스터/디스커버리 설정이 그대로 있는지 반드시 확인하세요. 기본 포트로 돌아가 있으면 클러스터에 합류하지 못합니다.

bash

```bash
sudo grep -E "http.port|transport|cluster.name|node.name|discovery|seed_hosts|initial_cluster_manager" /etc/opensearch/opensearch.yml
```

`opensearch-security/` 하위가 기본값으로 덮어써졌다면 백업에서 되돌립니다.

bash

```bash
sudo tar xzf /root/os-config-3.1.0-*.tar.gz -C / etc/opensearch/opensearch-security
sudo chown -R opensearch:opensearch /etc/opensearch
```

**D-4. 기동 및 합류 확인**

bash

<pre class="language-bash"><code class="lang-bash"><strong>sudo systemctl daemon-reload
</strong>sudo systemctl start opensearch
sudo journalctl -u opensearch -f
</code></pre>

다른 터미널에서:

bash

<pre class="language-bash"><code class="lang-bash"><strong>curl -sk -u admin:$PW "$OS/_cat/nodes?v&#x26;h=name,ip,version,node.role,master"
</strong></code></pre>

stage-logstackdb-01이 `3.3.1`으로 보이면 성공입니다.

**D-5. 할당 재활성화 후 green 대기**

bash

```bash
curl -sk -u admin:$PW -X PUT "$OS/_cluster/settings" \
  -H 'Content-Type: application/json' -d'
{ "persistent": { "cluster.routing.allocation.enable": null } }'

curl -sk -u admin:$PW "$OS/_cluster/health?wait_for_status=green&timeout=900s&pretty"
```

**green을 못 보면 다음 노드로 넘어가지 않습니다.**

***

#### E. 2번째: stage-logstackdb-03 (172.30.113.40)

D-1 \~ D-5를 stage-logstackdb-03 대상으로 그대로 반복합니다.

#### F. 3번째: stage-logstackdb-02 (172.30.113.39)

동일하게 반복합니다. 이 노드를 내리면 매니저가 stage-logstackdb-01 또는 stage-logstackdb-03로 넘어갑니다. 정상 동작이니 당황하지 마세요. 다만 이 시점에는 매니저 자격 노드가 2대만 남으므로(쿼럼 2) 반드시 나머지 두 대가 모두 살아 있는 상태여야 합니다.

***

#### G. 마무리

bash

```bash
# 전 노드 3.3.0 확인
curl -sk -u admin:$PW "$OS/_cat/nodes?v&h=name,ip,version,node.role,master"

# 상태 및 샤드
curl -sk -u admin:$PW "$OS/_cluster/health?pretty"
curl -sk -u admin:$PW "$OS/_cat/shards?v&h=index,shard,prirep,state,node" | grep -v STARTED
```

지연 설정을 원래대로 되돌립니다.

bash

```bash
curl -sk -u admin:$PW -X PUT "$OS/_all/_settings" \
  -H 'Content-Type: application/json' -d'{"settings":{"index.unassigned.node_left.delayed_timeout":null}}'
```

A단계에서 임시로 복제본을 올린 인덱스가 있다면 원래 값으로 되돌리세요.

마지막으로 Dashboards가 별도 서버에 있다면 같은 방식으로 3.3.0 RPM을 올립니다. 버전이 클러스터와 일치해야 합니다.





Dashboards는 **클러스터 3대가 모두 3.3.0으로 올라간 뒤에** 진행하세요. Dashboards가 상위 버전이고 클러스터가 하위 버전이면 접속이 거부됩니다. 파일 반입은 지금 미리 해 두셔도 됩니다.

#### 1. 외부망에서 다운로드

bash

```bash
BASE=https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/3.3.0

curl -O $BASE/opensearch-dashboards-3.3.0-linux-x64.rpm
curl -O $BASE/opensearch-dashboards-3.3.0-linux-x64.rpm.sig

sha512sum opensearch-dashboards-3.3.0-linux-x64.rpm > osd-checksum.txt
cat osd-checksum.txt
```

#### 2. scp 전송

Dashboards가 설치된 서버로만 보내면 됩니다. 어느 노드에 있는지 모르시면 각 서버에서 확인하세요.

bash

```bash
rpm -q opensearch-dashboards
systemctl status opensearch-dashboards --no-pager
```

전송:

bash

```bash
scp opensearch-dashboards-3.3.0-linux-x64.rpm osd-checksum.txt root@172.30.12.2XX:/tmp/
```

받는 쪽에서 검증:

bash

```bash
cd /tmp && sha512sum -c osd-checksum.txt
```

#### 3. 현재 설정 기록 (설치 전)

업그레이드 후 대조할 값들을 미리 남겨 둡니다.

bash

```bash
# 커스텀 플러그인 목록
sudo /usr/share/opensearch-dashboards/bin/opensearch-dashboards-plugin list

# 핵심 설정값
sudo grep -vE "^\s*#|^\s*$" /etc/opensearch-dashboards/opensearch_dashboards.yml
```

특히 `opensearch.hosts`(19200 포트), `server.host`, `server.port`, TLS 관련 경로, `opensearch_security.cookie.password`를 확인해 두세요.

#### 4. 설치

bash

```bash
sudo systemctl stop opensearch-dashboards
sudo cp -a /etc/opensearch-dashboards /root/osd-config-backup-$(date +%F)
sudo rpm -Uvh /tmp/opensearch-dashboards-3.3.0-linux-x64.rpm
sudo find /etc/opensearch-dashboards \( -name "*.rpmnew" -o -name "*.rpmsave" \)
```

`.rpmsave`가 나오면 설정이 기본값으로 밀려난 것입니다. 3번에서 기록한 값과 대조해 복원하세요.

bash

```bash
sudo diff /root/osd-config-backup-*/opensearch_dashboards.yml \
          /etc/opensearch-dashboards/opensearch_dashboards.yml
```

커스텀 플러그인을 설치했었다면 이 시점에 3.3.0용으로 재설치해야 합니다. 기본 번들 플러그인은 패키지에 포함되어 있으니 신경 쓰지 않아도 됩니다.

#### 5. 기동

bash

```bash
sudo systemctl daemon-reload
sudo systemctl start opensearch-dashboards
sudo journalctl -u opensearch-dashboards -f
```

Dashboards는 기동에 1\~3분 걸리고, 그동안 로그에 최적화 관련 메시지가 길게 나옵니다. `http server running` 또는 `Server running at` 메시지가 나오면 완료입니다.

#### 6. 확인

bash

```bash
curl -k -I https://localhost:15601
```

브라우저에서 로그인한 뒤 기존 대시보드, 인덱스 패턴, 저장된 검색이 그대로 뜨는지 확인하세요. 이 메타데이터는 클러스터의 `.kibana` 인덱스에 있으므로 Dashboards 재설치와 무관하게 유지됩니다.



#### 7. 에러 발생

```
Sep 21 10:51:19 stage-logstack-01 opensearch-dashboards[2752688]:  FATAL  Error: listen EACCES: permission denied 0.0.0.0:443
Sep 21 10:51:19 stage-logstack-01 systemd[1]: opensearch-dashboards.service: Main process exited, code=exited, status=1/FAILURE
Sep 21 10:51:19 stage-logstack-01 systemd[1]: opensearch-dashboards.service: Failed with result 'exit-code'.
Sep 21 10:51:19 stage-logstack-01 systemd[1]: opensearch-dashboards.service: Consumed 4.496s CPU time.
Sep 21 10:51:19 stage-logstack-01 systemd[1]: opensearch-dashboards.service: Scheduled restart job, restart counter is at 19.
Sep 21 10:51:19 stage-logstack-01 systemd[1]: Stopped "OpenSearch Dashboards".
Sep 21 10:51:19 stage-logstack-01 systemd[1]: opensearch-dashboards.service: Consumed 4.496s CPU time.
Sep 21 10:51:19 stage-logstack-01 systemd[1]: Started "OpenSearch Dashboards".
```

위 에러 발생시 아래와 같이 조치 진행

#### 해결: systemd 드롭인 (권장)

업그레이드에 영향받지 않는 방식입니다.

bash

```bash
sudo mkdir -p /etc/systemd/system/opensearch-dashboards.service.d

sudo tee /etc/systemd/system/opensearch-dashboards.service.d/override.conf > /dev/null <<'EOF'
[Service]
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
EOF

sudo systemctl daemon-reload
sudo systemctl reset-failed opensearch-dashboards
sudo systemctl start opensearch-dashboards
sudo journalctl -u opensearch-dashboards -f
```
