# Kafka
카프카 사용법 및 진행 상황입니다.

## Apache Kafka
> 오픈 소스 이벤트(메시지) 브로커  
  실시간 데이터 피드를 관리하기 위해 통일된 높은 처리량(대용량), 낮은 지연 시간을 지닌 플랫폼 제공  
{.is-info}
  
### 기존 서비스들의 구성
  * End-to-End 연결 방식의 아키텍처
  * 데이터 연동의 복잡성 증가 (HW, 운영체제, 장애 등)
  * 서로 다른 데이터 Pipeline 연결 구조
  * 확장이 어려운 구조    
    
### 카프카의 탄생 이유
  * **모든 시스템으로 데이터를 실시간으로 전송하여 처리할 수 있는 시스템**
  * **데이터가 많아지더라도 확장이 용이한 시스템**    
  
### 주요 스택	
- Kafka
- Kafka-Connect(Confluent)
- (추가 예정 ...)

### 생각해볼 내용
1. 현존하는 오픈소스 계열 큐잉 엔진 가운데서 가장 빠름
2. 아키텍처의 심플함(다운로드 후 실행하면 바로 사용가능)
3. 분산된 운영 환경에서 특정 노드가 다운 시 해당 문제에 대한 보장이 잘 되어있음
4. 카프카 엔진과 카프카 클라이언트 간에 상호의존성이 존재
	* 카프카 엔진과 실제 사용하는 클라이언트 간에 의존성 조율을 잘못하면 문제가 발생할 수 있음
	* 카프카 버전 등  
5. 카프카 기반 메시지 큐잉을 아예 하나의 독립적인 MS로 만드는 방법도 있음(퍼블리셔/컨슈머를 두고 데이터는 REST API/gRPC 등으로 통신)
6. 트랜잭션을 TOPIC으로 관리하는 방법도 있음
  
## Kafka 설치
Kafka 설치 및 운영은 윈도우보다 리눅스, 유닉스 계열에서 진행하길 권장  
(윈도우 설치 및 실행 시 실행 불가 에러 및 로그 초기화 관련 에러 발생)
- Kafka ([Download](https://kafka.apache.org/downloads), Binary downloads 권장)
- Kafka-Connect ([Kafka-Confluent Download](https://confluent.io))

## Kafka 명령어
### Kafka 로컬 실행 명령어  
* 윈도우 실행시 `$KAFKA_HOME\bin\windows\${file}.bat`로 경로를 설정해 실행 

1. zookeeper 서버 실행
```bash
$KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
```
2. kafka 서버 실행
```bash
$KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties
```

### Kafka Topic 명령어
현재 카프카 브로커(Broker)로 localhost:9092가 기동 중인 상태라 가정
```bash
# Topic 생성
$KAFKA_HOME/bin/kafka-topics.sh --create --topic ${topic_name} --bootstrap-server localhost:9092
# partition, replication 설정
$KAFKA_HOME/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 3 --topic ${topic_name}
# Topic 목록 확인
$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
# Topic 정보 확인
$KAFKA_HOME/bin/kafka-topics.sh --describe --topic ${topic_name} --bootstrap-server localhost:9092
# Topic 삭제(완전 삭제되었는지 확인 필요)
$KAFKA_HOME/bin/kafka-topics.sh --delete --bootstrap-server localhost:9092 --topic ${topic_name}

# Consumer Group 리스트
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
# Consumer Group 정보
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group ${group_name} --describe
# Consumer Group Offset 초기화 --to-earliest)
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group ${group_name} --topic ${topic_name} --reset-offsets --to-earliest --execute
# Offset 설정
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group ${group_name} --topic ${topic_name} --reset-offsets --to-offset 10 --execute
```  

### Kafka Topic 메시지 명령어
1. Producer
```bash
# 메시지 생산
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic ${topic_name} 
```
2. Consumer
```bash
# 메시지 소비
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ${topic_name}
# 메시지 소비 (감시모드)
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ${topic_name} --from-beginning
```
