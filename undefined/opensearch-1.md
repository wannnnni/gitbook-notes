# opensearch 조회 권한 부여

개발팀 dashboards viewer 권한 부여 방법

<pre><code><strong>security -> roles 추가
</strong><strong>dashboards_readonly
</strong><strong>index = *
</strong><strong>permissions = read
</strong><strong>
</strong><strong>
</strong><strong>chisviewuser에 권한 연결 (kibana_user, dashboards_readonly)
</strong><strong>
</strong><strong>opensearch-dashboards 서버의 opensearch-dashboards.yml 수정 후 재기동
</strong><strong>
</strong>opensearch_security.readonly_mode.roles: ["kibana_read_only","[신규정책]"]

</code></pre>

