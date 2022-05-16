# Logging/Monitoring
로깅, 모니터링 관련 자료입니다.

## ELK Stack의 구성
![elk_stack.png](/img/elk/elk_stack.png)

ELK는 위의 그림처럼 세 가지 기술로 구성된다.

* **Elasticsearch** : 로그 저장 및 검색 ([DownLoad](https://www.elastic.co/kr/downloads/elasticsearch))
* **Logstash** : 로그 수집 엔진 ([DownLoad](https://www.elastic.co/kr/downloads/kibana))
* **Kibana** : 로그 시각화 및 관리 ([DownLoad](https://www.elastic.co/kr/downloads/logstash))

좀 더 세부적으로는 아래와 같은 기능을 담당한다.

1. Data Processing (Logstash)  
- 서버 내의 로그, 웹, 메트릭 등 다양한 소스에서 데이터를 수집하여 입력  
- 데이터 변환 및 구조 구축  
- 데이터 출력 및 송신  
2. Storage (Elasticsearch)  
- 데이터 저장  
- 데이터 분석  
- 데이터 관리  
3. Visualize (Kibana)  
- Dashboard를 통한 데이터 탐색
- 팀원들과 공유 및 협업에 사용가능
- 엑세서 제어(Access Control) 사용 가능  

## ELK Stack의 장점
### 접근성 & 사용성
ELK는 [**Elastic**](https://www.elastic.co/kr/)이라는 회사에서 제공하는 **오픈소스**이다.
오픈소스를 내려받아 설치하는 것으로 구축이 완료됨. 
### 속도
ELK중 데이터 보관 및 분석 역할을 담당하는 Elasticsearch는 거의 실시간(Real-Time)에 가깝게 데이터 처리 가능
### 유연성
ELK는 각자의 기능을 담당하는 모듈을 붙여서 구성을 한다. 그렇기 때문에 그 기능을 담당할 수 있다면 다른 모듈로 대체 가능하다.  
(ex. Logstash를 사용하지 않고 다른 로그 수집기로 대체 가능)

## ELK Stack의 최근 동향
### Beats의 도입
기존 ELK에서 가장 큰 문제점 중 하나는 Logstash였다.
데이터 수집을 하는 Logstash는, 원하는 형태로의 데이터 입출력 변환 기능까지 맡고 있어 오버헤드가 컸다.  
그 결과 Elastic에서는 오로지 데이터의 수집만을 담당하는 경량화된 모듈 Beats를 도입했다.  
**Beats**는 한 가지를 이르는 말이 아니라, 로그파일, 메트릭, 네트워크 데이터, Windows 이벤트 로그 ,가동 시간 모니터링 등 수집하고자 하는 데이터의 유형별로 모듈이 존재한다.  
우리 MSA 프로젝트에서는 로그 파일을 주로 다룰 것이기 때문에 Beats를 사용하게 된다면 FileBeat라는 로그 수집기를 사용할 예정이다.
- **FileBeat** : 로그 수집기 ([DownLoad](https://www.elastic.co/kr/downloads/beats/filebeat))  

## MSA ELK 구상도
### 기본 구성
- LogFile -> Logstash -> Elasticsearch <- Kibana  

![elk_basic.png](/img/elk/elk_basic.png)  

### Filebeat를 추가한 구성
- LogFile -> Filebeat -> Logstash -> Elasticsearch <- Kibana  
- Filebeat를 추가해 Logstash의 오버헤드를 줄이는 구성

![elk_filebeat.png](/img/elk/elk_filebeat.png)  

### 이외 가능한 구성
1. 큐(Queue)를 이용한 로그 전달 구성 (Queue의 위치를 어디로 정할지는 생각할 요소)
- LogFile -> Filebeat -> (Queue) -> Logstash -> (Queue) -> Elasticsearch <- Kibana

Kafka 를 도입하는 많은 이유 중 하나는 트래픽이 몰리면 Logstash, Elasticsearch 만으로는 부하를 견디기 힘들다고한다.  
운영 상 로그를 남길때 Elasticsearch가 꺼져있다면 로그가 전달되지 못함,  
전달되지 못한 로그는 Buffer가 가지게되는 경우가 발생하고 Filebeat와 Logstash가 해당 역할에서 부족한 부분이 있음.  
*) 일반적으로 최소 주키퍼 3대, 카프카 3대로 구성  

## Elasticsearch
### 실행방법
1. 기본 포트 : 9200, 9300
```bash
$ELASTICSEARCH_HOME/bin/elasticsearch.sh	# Elasticsearch 실행

# elasticsearch 사용 예제
curl -XGET 'localhost:9200/_cat/indices?v'	# index 목록 조회
curl -XPUT 'localhost:9200/customer?pretty' 
curl -XGET 'localhost:9200/customer2/_search?pretty'	# index 내용 조회
curl -XDELETE 'localhost:9200/customer2/info/1?pretty'	# index내 정보 삭제
curl -XDELETE 'localhost:9200/customer2'
```  
2. **elasticsearch.yml** 설정 파일(간단 세팅)
```yml
network.host: 0.0.0.0	# 호스트 주소 설정
http.port: 9200				# 포트 설정
```

## Logstash
### 실행방법
logstash는 로그 수집과 elasticsearch로 가는 정보의 포맷을 수정 가능함
1. 기본포트 : 5044, 9600  
```bash
# logstash 실행
$LOGSTASH_HOME/bin/logstash.sh -f $LOGSTASH_HOME/config/logstash.conf 	# linux
$LOGSTASH_HOME\bin\logstash.bat -f $LOGSTASH_HOME\config\logstash.conf	# windows
```
2. **logstash.conf** 설정 파일(간단 세팅)
```conf
input {
    tcp {
        # host => "13.124.248.164" # 호스트 설정
        port => 4560
        codec => json_lines # JSON 설정
    }
}

# 내보낼 데이터 필터링
filter {
    json {
        source => "message"
    }
}

output {
		# elasticsearch 정보 설정
    elasticsearch {
        # hosts => ["http://13.124.248.164:18000"]
        hosts => ["http://localhost:9200"]
        index => "logstash-%{+YYYY.MM.dd}"
        ecs_compatibility => disabled
    }
    # CUI 확인을 위한 설정
    stdout {
        codec => rubydebug
    }
}
```
## Kibana
### 실행방법
kibana는 elasticsearch로부터 얻은 데이터를 다양한 방법으로 시각화 가능
1. 기본포트 : 5601
```bash
# logstash 실행
$KIBANA_HOME/bin/kibana.sh
```
2. kibana.yml 설정 파일(간단 설정)
```yml
server.host: "0.0.0.0"		# kibana 호스트 설정
server.port: 18001
server.shutdownTimeout: "5s"
#elasticsearch.hosts: [ "http://elasticsearch:9200" ]
elasticsearch.hosts: [ "http://localhost:9200" ]	# elasticsearch 호스트 설정
monitoring.ui.container.elasticsearch.enabled: true
```

## Filebeat
### 실행방법
- filebeat는 각 프로젝트마다 로그파일을 수집함
```bash
# filebeat 실행
systemctl start filebeat
systemctl status filebeat
```
- **filebeat.yml** 설정(간단 설정)
```yml
# ============================== Filebeat inputs ===============================
filebeat.inputs:

- type: log
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - C:\Intellij\user-service\logs\*-json.log
    - C:\work\demo\logs\*-json.log
    # - C:\Intellij\order-service\logs\*-json.log
    # - C:\Intellij\catalog-service\logs\*-json.log
  fields:
  ### Multiline options
  # The regexp Pattern that has to be matched. The example pattern matches all lines starting with [
  # 해당 패턴을 만나야 다음 라인으로 인식하는듯 함 (현재 내 로그는 스프링부트 로그이기 떄문에 날짜 패턴으로 라인을 구분하도록 설정)
  multiline.pattern: ^[0-9]{4}-[0-9]{2}-[0-9]{2}[[:space:]][0-9]{2}:[0-9]{2}:[0-9]{2}.[0-9]{3}[[:space:]]
  multiline.negate: true
  multiline.match: after
- type: filestream
  enabled: false
  paths:
    - /var/log/*.log
# ============================== Filebeat modules ==============================
filebeat.config.modules:
  # Glob pattern for configuration loading
  path: ${path.config}/modules.d/*.yml
  # Set to true to enable config reloading
  reload.enabled: false
  # Period on which files under path should be checked for changes
  #reload.period: 10s
# ======================= Elasticsearch template setting =======================
setup.template.settings:
  index.number_of_shards: 1
  #index.codec: best_compression
  #_source.enabled: false
# =================================== Kibana ===================================
# setup.kibana:
  # 모니터링을 위해 바로 키바나로 내보내는 설정 시 작성
  # host: "localhost:5601"
# ================================== Outputs ===================================
# ---------------------------- Elasticsearch Output ----------------------------
# output.elasticsearch:
  # 엘라스틱서치에 직접 데이터 전송시 설정하는 부분
  # hosts: ["localhost:9200"]
# ------------------------------ Logstash Output -------------------------------
output.logstash:
  # The Logstash hosts 
  # logstash와 연결하는 설정
  hosts: ["localhost:5044"]
# ================================= Processors =================================
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
# etc options...
```

