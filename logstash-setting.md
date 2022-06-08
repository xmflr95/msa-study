# Logstash Setting
Logstash filter 세팅 기록 자료입니다.  

#### 구성
Filebeat - **Kafka** - Logstash - Elasticsearch - Kibana  
Kafka를 Filebeat와 Logstash사이에 배치함으로써 Logstash의 부하를 막고 높은 **확장성**을 가져갈 수 있게됨.  
(+ Logstash의 경우 데이터 parsing도중 에러 발생 시 최악의 경우, **데이터 손실**이 발생, 그 이전에 Kafka를 두어 신뢰성을 보장하도록함.)

## FileBeat
Filebeat를 **Producer**로 사용하도록 세팅
```yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\log\tda-iam.json*.log
    - C:\log\tda-tcn.json*.log
  
  # multiline 설정
  multiline.pattern: '^{'
  multiline.negate: true 
  multiline.match: after
# settings...
# -- Kafka Output ---
output.kafka:
  hosts: ["localhost:9092"]
  # codec.format:
  #   string: '%{[message]}'
  topic: "logtest"
  partition.round_robin:
    rechable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
# settings...
```

## Logstash
Logstash를 **Consumer**로 사용되도록 세팅
```ruby
input {
  # tcp settings
  # tcp {
  #   port => 4560
  #   codec => json_lines
  # }
  # kafka subscribe (카프카 구독 설정)
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => ["logtest"] # topic
    consumer_threads => 2
    group_id => "group-log" # group
  }
  # beats setting
  # beats {
  #   port => 5044
  # }
  # jdbc 직접 연결
  # jdbc {
  #   jdbc_driver_library => "C:\dev\logstash-7.12.0\tools\mysql-connector-java-5.1.45-bin.jar"
  #   jdbc_validate_connection => true
  #   jdbc_driver_class => "com.mysql.jdbc.Driver"
  #   jdbc_connection_string => "jdbc:mysql://localhost:3306/board"
  #   jdbc_user => "root"
  #   jdbc_password => "root"
  #   statement => "select * from board.post"
  #   schedule => "* * * * *"
  # }
}

filter {
  json {
    source => "message"
    target => "doc"
    remove_field => ["message"]
  }
  json {
    source => "[doc][message]"
    target => "logs"
    remove_field => ["[doc]"]
  }
  mutate {
    add_field => {
      "timestamp" => "%{[logs][@timestamp]}"
      "severity" => "%{[logs][severity]}"
      "service" => "%{[logs][service]}"
      "trace" => "%{[logs][trace]}"
      "span" => "%{[logs][span]}"
      "parent" => "%{[logs][parent]}"
      "exportable" => "%{[logs][exportable]}"
      "pid" => "%{[logs][pid]}"
      "thread" => "%{[logs][thread]}"
      "class" => "%{[logs][class]}"
      "rest" => "%{[logs][rest]}"
    }

    remove_field => ["[logs]"]
  }

  # grok 문법으로 filter링 가능
  # grok {
  #   match => { "message" => "%{COMBINEDAPACHELOG}" }
  # }
  # date {
  #   match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  # }

  # json simple parsing
  # json {
  #   source => "message"
  #   remove_field => ["message", "host", "agent", "input", "ecs", "@version"]
  #   # remove_field => ["message", "host", "agent", "input", "ecs", "@version", "log", "tags"]
  # }
  # json {
  #   source => "message"
  #   remove_field =>  ["host", "agent", "log", "input", "ecs"]
  # }

  # 불필요 필드 삭제
  # mutate {
  #   remove_field => ["host", "agent"]
  # }
  
  # 조건문 사용
  # if [message] =~ "\tat" {
  #   grok {
  #     match => ["message", "^(\tat)"]
  #     add_tag => ["stacktrace"]
  #   }
  # }
  # dissect {
  #   mapping => {
  #     "message" => "%{log_time}|%{level}|%{logger}|%{thread}|%{msg}"
  #   }
  # }
  # date{
  #   match => ["log_time", "yyyy-MM-dd HH:mm:ss.SSS"]
  #   target => "@timestamp"
  # }

  # spring boot 기본 로그 파싱
  # grok {
  #   match => { "message" => "%{TIMESTAMP_ISO8601:[loginfo][date]}\s+%{LOGLEVEL:[loginfo][level]} %{POSINT:[loginfo][pid]} --- \[\s*%{DATA:[loginfo][thread]}\] %{DATA:[loginfo][class]}\s+: %{GREEDYDATA:[loginfo][message]}" }
  # }
  # date {
  #   match => ["[loginfo][date]", "yyyy-MM-dd HH:mm:ss.SSS"]
  #   target => "@timestamp"
  #   timezone => "Asia/Seoul"
  # }
  # mutate {
  #   remove_field => ["host", "agent", "message"]
  # }
}

output {
  # elasticsearch로 보내는 설정
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logkafka-%{+YYYY.MM.dd}"
    ecs_compatibility => disabled
  } 
  stdout { codec => rubydebug } 
}
```

## 진행과정 상세
filebeat에서 json으로 내보내진 로그를 읽어 Kafka로 보내면 아래와 같은 데이터가 Kafka에 "message"로 보내짐  
불필요한 데이터가 많기 때문에 필터링이 필요함

```txt
{"@timestamp":"2022-05-16T05:52:45.529Z","@metadata":{"beat":"filebeat","type":"_doc","version":"7.12.0"},"message":"{\"@timestamp\":\"2022-05-16T05:52:45.324Z\",\"severity\":\"INFO\",\"service\":\"tda-iam\",\"trace\":\"\",\"span\":\"\",\"parent\":\"\",\"exportable\":\"\",\"pid\":\"3108\",\"thread\":\"RMI TCP Connection(6)-127.0.0.1\",\"class\":\"com.netflix.discovery.DiscoveryClient\",\"rest\":\"Completed shut down of DiscoveryClient\"}","log":{"offset":79242,"file":{"path":"C:\\log\\tda-iam.json.log"}},"input":{"type":"log"},"ecs":{"version":"1.8.0"},"host":{"name":"DESKTOP-EECQ664","architecture":"x86_64","os":{"name":"Windows 10 Home","kernel":"10.0.19041.1706 (WinBuild.160101.0800)","build":"19044.1706","type":"windows","platform":"windows","version":"10.0","family":"windows"},"id":"bca4dce9-1abb-4cdf-b1d3-a86fda32b83c","ip":["fe80::897c:9f06:48dd:2a8a","192.168.56.1","fe80::e576:b64b:f48d:d02e","169.254.208.46","fe80::7851:8578:7700:27f4","169.254.39.244","fe80::3dbb:bf5f:efff:6c02","169.254.108.2","fe80::dcaf:80bb:b9ec:f26b","192.168.2.220","fe80::c09e:8cad:253b:20a7","172.31.0.1"],"mac":["0a:00:27:00:00:34","08:71:90:27:e8:fe","08:71:90:27:e8:ff","0a:71:90:27:e8:fe","00:e0:4c:36:1d:da","00:15:5d:ff:04:8a"],"hostname":"DESKTOP-EECQ664"},"agent":{"id":"3ca32012-e0da-4a62-b918-7f503dd7a695","name":"DESKTOP-EECQ664","type":"filebeat","version":"7.12.0","hostname":"DESKTOP-EECQ664","ephemeral_id":"c8dad804-a3ba-47a7-9a64-7b077cda955b"}}
```

```ruby
json {
  source => "message"
  target => "doc"
  remove_field => ["message"]
}
```
logstash에서 kafka에게 구독해서 받은 데이터는 "message" 문자열로 전송되기 때문에 해당 부분을 logstash json 필터를 이용하여 파싱함.  
1. `source`를 이용해 message를 json 형태로 파싱함
2. `target`으로 해당 데이터의 최상단에 "doc"이라 지정한 필드에 파싱값이 저장되도록 함  
(생략 가능하나 파싱된 필드값이 남겨야할 필드와 이름이 같을 경우 덮어쓰기가 되어버려 주의가 필요)
3. `remove_field`를 이용해 기존의 "message" 필드를 제거하여 "doc" 필드만 남도록 함  

결과로 JSON값인 "doc" 필드를 얻을 수 있음  
하지만 파싱만 되었을뿐 위의 불필요한 값을 제거하는 작업이 필요함

```ruby
json {
  source => "[doc][message]"
  target => "logs"
  remove_field => ["[doc]"]
}
```
1. `source => [doc][message]`를 이용해 "doc"필드의 "message"값을 다시 JSON 파싱함([doc][message]가 json string일 경우)
2. 해당 내용은 "logs"라는 필드로 최상단에 적용되도록 지정
3. 불필요한 내용을 담은 "doc"필드는 삭제

> 결과
```ruby
{
  "@version" => "1",
  "@timestamp" => 2022-05-16T05:20:37.261Z,
  "logs" => {
    "pid" => "11364",
    "rest" => "Resolving eureka endpoints via configuration",
    "parent" => "",
    "thread" => "AsyncResolver-bootstrap-executor-0",
    "class" => "c.n.d.s.r.aws.ConfigClusterResolver",
    "span" => "",
    "exportable" => "",
    "severity" => "INFO",
    "service" => "tda-iam",
    "trace" => "",
    "@timestamp" => "2022-05-16T05:20:37.261Z"
  }
}
```
결과로 "logs" 필드를 얻게되었고 이대로 사용해도 무방해보이나 해당 필드값을 각각의 필드로 분리하여 사용하는게 좋아보여 분리 작업을 진행함

```ruby
mutate {
  add_field => {
    "timestamp" => "%{[logs][@timestamp]}"
    "severity" => "%{[logs][severity]}"
    "service" => "%{[logs][service]}"
    "trace" => "%{[logs][trace]}"
    "span" => "%{[logs][span]}"
    "parent" => "%{[logs][parent]}"
    "exportable" => "%{[logs][exportable]}"
    "pid" => "%{[logs][pid]}"
    "thread" => "%{[logs][thread]}"
    "class" => "%{[logs][class]}"
    "rest" => "%{[logs][rest]}"
  }

  remove_field => ["[logs]"]
}
```
1. "logs" 필드값들을 분리하기 위해 `add_field`로 각각 필드값을 재선언 한 후 "logs" 필드들의 값을 할당함
2. "logs" 필드 삭제

> 결과
```ruby
{
  "@version" => "1",
  "class" => "c.n.d.s.r.aws.ConfigClusterResolver",
  "severity" => "INFO",
  "thread" => "AsyncResolver-bootstrap-executor-0",
  "rest" => "Resolving eureka endpoints via configuration",
  "exportable" => "",
  "@timestamp" => 2022-05-16T06:45:51.180Z,
  "pid" => "22408",
  "trace" => "",
  "span" => "",
  "parent" => "",
  "service" => "tda-tcn",
  "timestamp" => "2022-05-16T06:45:38.527Z"
}
```
위와 같이 깔끔한 로그 값을 얻을 수 있고 Kibana에서도 편하게 확인 가능하다.  

###### 진행할 때 발생했던 상황
1. `"timestamp" => "%{[logs][@timestamp]}"`로 `@timestamp` 필드를 재선언하려했으나 `@timestamp`는 date값이 아니면 문자열로는 할당이 불가능하고 ruby코드를 이용해 변환 후 할당이 가능한듯 보임, 따라서 이번 경우엔 "timestamp"로 선언 후 저장시킴.  
2. 임의로 `@timestamp` 필드를 선삭제 후 할당할 경우 elasticsearch로 보내는 index에 적용될 시스템 time값이 삭제되어 elasticsearch index값 "logkafka-%{+YYYY.MM.dd}"에서 뒤의 date값이 사라짐,   
꼭 삭제가 필요한 경우 먼저 `mutate`에서 `add_field`를 이용해 "[@metadata][field_name]"으로 선언 후 index값에 해당 값을 사용해야함  

> 이후 추가 사항  

logstash에서 elasticsearch로 데이터 전송 시 중복 수집되는 경우가 발생(정확한 원인은 찾아야함).
logstash의 fingerprint 필터와 elasticsearch 전송시 `document_id`를 지정하는 것으로 중복 제거는 가능함
```ruby
fingerprint {
  method => "SHA256"
  source => ["timestamp", "rest"]
  target => "fingerprint" # _source fields에 항목 추가됨
  concatenate_sources => true
}
# ... settings ...
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logkafka-%{+YYYY.MM.dd}"
    document_id => "%{fingerprint}"  # 중복 제거(Deduplication)
    ecs_compatibility => disabled
  } 
  stdout { codec => rubydebug } 
}
```