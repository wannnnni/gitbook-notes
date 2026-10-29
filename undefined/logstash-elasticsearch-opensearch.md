# logstash (elasticsearch→opensearch)

```
vi logstash/config/logstash.config

output {
  #stdout {   codec => rubydebug   }
  if "InOut" in [LogCategory]{
    opensearch {
      hosts               => ["${OPENSEARCH}"]
      index               => "%{[type]}_%{+YYYY.MM.dd}"
      action              => "update"
      doc_as_upsert       => true
      document_id         => "inout_%{ContextId}_%{CreateAt}"
      user                => "opensearch"
      codec               => "json"
      password            => "caretech12!@"
    }
  }
  else {
    opensearch {
      hosts               => ["${OPENSEARCH}"]
      index               => "%{[type]}_%{+YYYY.MM.dd}"
      user                => "opensearch"
      codec               => "json"
      password            => "caretech12!@"
    }
  }
}
```

```
vi logstash/Dockerfile -> opensearch plugin install 추가

FROM docker.elastic.co/logstash/logstash:7.16.2
USER root
# logstash
RUN rm -f /usr/share/logstash/pipeline/logstash.conf
# opensearch plugin install
RUN /usr/share/logstash/bin/logstash-plugin install logstash-output-opensearch
COPY config/ /usr/share/logstash/config/

# cron
RUN yum -y update
RUN yum -y install cronie
RUN yum -y install yum-cron
# RUN touch /var/log/cron.log
COPY crontab /etc/cron.d/cjob
RUN chmod 0644 /etc/cron.d/cjob
RUN crontab /etc/cron.d/cjob

# -n 포그라운드로 실행
CMD crond && /usr/share/logstash/bin/logstash -f config/logstash.config

```

```
# Docker image 생성
sudo docker build -t stg/chislog:1.0 .

# Docker image 확인
sudo docker images | grep stg/chislog
```

```
# 젠킨스 접속
STG-Test-Backend
매개변수 추가 (chislog)

# docker tag 값 변경

#repositories name 설정
repositories="stg/${domain}"
output=$(/usr/local/bin/aws ecr describe-repositories --query "repositories[?repositoryName=='$repositories']" --profile awsstg  | jq '.[].registryId' -r)
#echo $output

#repositories가 존재하는 경우 pass
if [ $output == "637423521246" ]; then
	echo exist
#repositories가 없는 경우 새로 생성
else
	echo repositories create
    sudo /usr/local/bin/aws ecr create-repository --repository-name $repositories --profile awsstg
fi
    /usr/local/bin/aws ecr get-login-password --region ap-northeast-2 --profile awsstg | sudo docker login --username AWS --password-stdin 637423521246.dkr.ecr.ap-northeast-2.amazonaws.com
    sudo docker tag "stg/chislog:1.0" "637423521246.dkr.ecr.ap-northeast-2.amazonaws.com/$repositories:1.0"
    sudo docker push "637423521246.dkr.ecr.ap-northeast-2.amazonaws.com/$repositories:1.0"
```
