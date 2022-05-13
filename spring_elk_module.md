# ELK Module
elasticsearch를 어떻게 spring boot에 적용할지 구상
## Dependency
Elasticsearch 쿼리 조회를 위한 Maven(혹은 Gradle 외) 의존성 추가  
**현재 사용중인 Elasticsearch와의 버전을 맞추는 것이 중요함**
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
  	<!-- 현재 사용중인 Elasticsearch 버전을 적용할 것 -->
    <version>7.12.0</version> 
</dependency>
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>7.12.0</version>
</dependency>
```