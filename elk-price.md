# ELK 가격 정책 조사

## 라이선스 변경 주의 사항(2021.01.15 이후)

Elasticsearch 개발사인 Elastic과 AWS 간의 서비스 제공에서 갈등으로 인해
Elasticsearch, Kibana 등 Elastic 사의 제품 라이선스가 변경됨

- 바이너피 배포판 : 변경 없음
- 소스코드
  - 7.10까지의 Free OSS 버전은 Apache-2.0 라이선스, 나머지는 Elastic License 적용
  - 7.11의 릴리즈와 더불어 모든 버전에 대해, Free OSS 타입이 SSPL과 Elastic License의 듀얼 라이선스 체제로 변경(택 1)
- 영향도
  - Elastic Cloud 사용자 혹은 직접 다운 받아 사용하는 사용자는 영향 없음
  - Elasticsearch 또는 Kibana의 소스코드를 가지고 자체 서비스를 제작해 남들에게 제공하는 경우 영향이 있음
    - 소스코드 변경 및 릴리즈를 모두 공개해야함

* OSI에서는 SSPL 라이선스는 오픈 소스가 아님을 밝힘(소스 수정 시 그와 관련된 모든 소스 공개필요)
  * MongoDB는 회사 내부에서만 사용하면 문제가 없다.
  * MongoDB를 외부(제 3자)에 서비스로 제공하는 경우, 서비스 소스코드 전부를 공개해야 한다.

* Elastic License 
* 상업적 사용 및 서비스 시 소스코드 공개의무 등이 발생할 수 있으며, 사용 및 서비스를 위해 Elastic과 협의가 필요할 수 있음
* Elastic License는 SSPL과는 달리 매우 간단하게 작성이 되었습니다. 먼저 기존 오픈소스 라이선스가 가지는 소스 코드 수정, 사용, 배포 등에 대해서는 거의 모든 자유를 허용, 다만 아래와 같은 제약사항이 있음
  * 제품을 다른 사람에게 관리형 서비스로 제공할 수 없습니다. 
  * 라이선스 키 기능을 우회하거나 라이선스 키로 보호되는 기능을 제거하면/숨길 수 없습니다. 
  * 라이선스, 저작권 또는 기타 통지를 제거하거나 숨길 수 없습니다.

> 가격문의 결과
> 전문가의 측정이 필요하나 유료 라이선스 구매시 대략적으로 1노드 당 $7,000 책정 중(2022.10.24) 해당 가격도 사용량을 감안하지 않은 단순 가격.
> 연간 구독으로 운영되는 라이선스, 다년 계약은 가능함
> 최소 3노드로 운영해야 라이선스를 구매 가능한 것으로 보임(연 3 * $7,000 = $21,000)

Elastic License는 사실상 클라우드 서비스 제공 업체에서의 사용은 금지하도록 함(AWS 에서 제공하는 Elasticsearch와 같은 서비스), 무상사용(Strip-mining)을 막음
이에 AWS는 Elasticsearch 서비스를 계속하기 위해 Elasticsearch를 Fork했고, 이를 Open Distro for Elasticsearch라고 명명하며 Apache License 2.0을 적용하고, 커뮤니티를 운영 중  
 
참고 Link
* https://www.elastic.co/kr/pricing/faq/licensing
* https://www.elastic.co/kr/licensing/elastic-license/faq
* https://aws.amazon.com/ko/opensearch-service/the-elk-stack/what-is-opensearch/
* https://sktelecom.github.io/blog/2021/20210328-elasticlicense/
* https://tech.kakao.com/2021/09/08/opensource-license/#elementor-toc__heading-anchor-12
* http://kimjmin.net/2020/06/2020-06-elastic-devrel/

## ElasticSearch
### 라이선스
- 7.11 릴리즈 이후 : SSPL / Elastic License V2(ELv2) (중 택1)
- 7.10 이전 Free OSS 버전일 경우 : Apache-2.0 라이선스
### X-Pack
X-Pack은 보안, 알림, 모니터링, 보고, 그래프 기능을 설치하기 편리한 단일 패키지로 번들 구성한 Elastic Stack 확장 프로그램이다.  
X-Pack은 모니터링, 보고, 경고, 보안 및 기타 여러 기능과 같은 기능을 제공하는 ELK 스택의 확장, 설치된 ELK 스택과 버전과 같은 X-Pack이 설치되어야함  
* 무료 Elasticsearch License를 사용하는 경우 표준 X-Pack 기능에 액세스 가능, 모든 기능을 사용하려면 구독이 필요
* Elasticsearch License를 사용하기 때문에 관리형 서비스 및 상업적 용도, 소스 변경 빌드 배포 시 Elastic사와 협의 혹은 소스 코드 공개가 필요함

### 공식사이트 라이센스별 지원기능 구분 (ElasticSearch, Kibanba)
https://www.elastic.co/kr/subscriptions

### 비용 구분(온프레미스 구독, 무료 개방형 Elastic(ELK) Stack)
(Basic 기준)
#### 1. 무료 기능
1. X-Pack 무료
   - Security(6.8.0, 7.1.0 이후 버전)
2. 시각화 관련 기능
   - DashBoard (스택 로깅, 시각화 대시보드) 기본 제공  
3. Monitoring
   - ELK 스택 모니터링

#### 2. 유료 기능
X-Pack의 유료 기능들(플러그인) 외
1. Kibana 알람 기능(Alerting) 플러그인
   - 추후 로그 분석에 따른 장애 알람을 추가할 경우 필요  
   - 기본(Basic)버전은 Kibana 경보 기본 기능만 사용가능
   - 시각화와 별개로 로그 분석을 통해 빠른 장애 피드백이 필요하면 필수적인 기능
   - 기술 도입 혹은 연구해본 기업 예시
     - 배달의 민족 : https://techblog.woowahan.com/2659/
     - 29CM(무신사) : https://medium.com/29cm/open-distro-for-elasticsearch-%EB%A1%9C%EA%B7%B8%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%9E%A5%EC%95%A0-%ED%83%90%EC%A7%80-8ff60dc3e5f
     - 데일리호텔 : https://www.theteams.kr/teams/865/post/64570
2. 스택 모니터링
   - 기본(Basic) 버전은 멀티 스택 모니터링은 불가
3. 보고(Report) 기능
   - 보고서(PDF&PNG 유료 라이선스, CSV는 가능) 키바나 화면을 이용한 생성, 출력 가능
4. 이상 징후 감지
   - 시계열 데이터 비교, AI ops(로그 속도 급증 설명) 등 이상 징후 감지 기능 이용가능 
5. 머신러닝 관련 기능
   - 머신러닝 관련 시각화 기능은 Data Visualizer 제외 유료 서비스 
6. 데이터 프레임 분석
   - 기능항목 : 이상값 탐지, 회귀, 분류, 기능 중요도
7. Elastic사의 기술지원

> 대체재  

Elasitc 사에서 제공하는 X-Pack 플러그인의 기능들이 유료 서비스가 되면서  
Alert을 비롯한 쓸만한 기능들은 Basic 에서 모든 기능의 사용은 불가능해짐(라이선스 필요)  
대안으로 Amazon AWS에서 엘라스틱서치용 오픈소스 플러그인으로 제공하는 Open Distro를 사용할 수 있음(배민, 29CM(무신사) 등)  
참고 Link : https://opendistro.github.io/for-elasticsearch-docs/docs/install/plugins/

## Kibana


## Logstash
Logtash의 경우, Apache License 2.0을 따르나 x-pack 폴더에서 사용 시에는 Elastic License가 적용됩니다. (X-Pack 이용시 Elastic License 적용)  

Fluentd를 사용하는 경우도 있음

## FileBeat (Beats)
경량 로그 수집

## 라이센스 무료 전환때 나온 메시지 기록
Some functionality will be lost if you replace your TRIAL license with a BASIC license. Review the list of features below.
Watcher will be disabled
Logstash will no longer poll for centrally-managed pipelines
Security will default to disabled (set xpack.security.enabled to enable security).
Cross-Cluster Replication will be disabled
Beats will no longer be able to use centrally-managed configuration
Multi-cluster support is disabled for clusters with [BASIC] license. If you are running multiple clusters, users won't be able to access the clusters with [BASIC] licenses from within a single X-Pack Kibana instance. You will have to deploy a separate and dedicated X-pack Kibana instance for each [BASIC] cluster you wish to monitor.
Graph will be disabled
Machine learning will be disabled
JDBC and ODBC support will be disabled, but you can continue to use SQL CLI and REST endpoint


TRIAL 라이선스를 BASIC 라이선스로 교체하면 일부 기능이 손실됩니다. 아래 기능 목록을 검토하세요.
감시자가 비활성화됩니다.
Logstash는 더 이상 중앙에서 관리되는 파이프라인을 폴링하지 않습니다.
보안은 기본적으로 비활성화되어 있습니다(보안을 활성화하려면 xpack.security.enabled 설정).
클러스터 간 복제가 비활성화됩니다.
Beats는 더 이상 중앙 관리 구성을 사용할 수 없습니다.
[BASIC] 라이센스가 있는 클러스터에서는 다중 클러스터 지원이 비활성화됩니다. 여러 클러스터를 실행 중인 경우 사용자는 단일 X-Pack Kibana 인스턴스 내에서 [BASIC] 라이선스로 클러스터에 액세스할 수 없습니다. 모니터링하려는 각 [BASIC] 클러스터에 대해 별도의 전용 X-pack Kibana 인스턴스를 배포해야 합니다.
그래프가 비활성화됩니다
머신 러닝이 비활성화됩니다.
JDBC 및 ODBC 지원이 비활성화되지만 SQL CLI 및 REST 끝점을 계속 사용할 수 있습니다.