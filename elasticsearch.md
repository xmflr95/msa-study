# Elasticsearch Query
Elasticsearch Query 조사 자료입니다.  
(정리에 순서가 없습니다.)

## Full Text Query
### match_all
해당하는 인덱스의 모든 도큐먼트를 검색
```js
// no query
GET logkafka-2022.05.16/_search
// match_all query run
GET logkafka-2022.05.16/_search
{
  "query": {
    "match_all": {}
  }
}
```
### match
해당하는 필드에 값이 있는 모든 문서 검색
```js
GET cafe/_search
{
  "query": {
    "match": {
      "title": "iced"
    }
  }
}
```
> 실행결과
```json
"hits" : {
  "total" : {
    "value" : 3,
    "relation" : "eq"
  },
  "max_score" : 0.14874382,
  "hits" : [
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "KwlS_X8BQ3yq82ANLq4s",
      "_score" : 0.14874382,
      "_source" : {
        "title" : "iced americano",
        "price" : "1500"
      }
    },
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LAlS_X8BQ3yq82ANoq7k",
      "_score" : 0.12703526,
      "_source" : {
        "title" : "iced cafe latte",
        "price" : "2500"
      }
    },
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LQlS_X8BQ3yq82ANuK5y",
      "_score" : 0.12703526,
      "_source" : {
        "title" : "iced cafe moca",
        "price" : "3000"
      }
    }
  ]
}
```
위의 결과를 보면 'iced'가 들어간 모든 결과가 조회되는 것을 볼 수 있다.  
`match`의 검색 조건은 기본적으로 **OR**이다. **AND**를 사용하고 싶은 경우엔 `operator`옵션을 추가한다.

```js
// 'iced' or 'cafe' 검색
GET cafe/_search
{
  "query": {
    "match": {
      "title": "iced cafe"
    }
  }
}
```
> 실행결과
```json
"hits" : {
  "total" : {
    "value" : 3,
    "relation" : "eq"
  },
  "max_score" : 0.57417387,
  "hits" : [
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LAlS_X8BQ3yq82ANoq7k",
      "_score" : 0.57417387,
      "_source" : {
        "title" : "iced cafe latte",
        "price" : "2500"
      }
    },
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LQlS_X8BQ3yq82ANuK5y",
      "_score" : 0.57417387,
      "_source" : {
        "title" : "iced cafe moca",
        "price" : "3000"
      }
    },
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "KwlS_X8BQ3yq82ANLq4s",
      "_score" : 0.14874382,
      "_source" : {
        "title" : "iced americano",
        "price" : "1500"
      }
    }
  ]
}
```
`_score`의 변화가 있어 우선 검색 순위가 달라졌을뿐 결과물은 같다.
아래는 **AND**연산을 이용해 조회하는 방법이다.
```js
GET cafe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "iced cafe",
        "operator": "and"
      }
    }
  }
}
```
> 실행결과
```json
"hits" : {
  "total" : {
    "value" : 2,
    "relation" : "eq"
  },
  "max_score" : 0.57417387,
  "hits" : [
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LAlS_X8BQ3yq82ANoq7k",
      "_score" : 0.57417387,
      "_source" : {
        "title" : "iced cafe latte",
        "price" : "2500"
      }
    },
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LQlS_X8BQ3yq82ANuK5y",
      "_score" : 0.57417387,
      "_source" : {
        "title" : "iced cafe moca",
        "price" : "3000"
      }
    }
  ]
}
```
실제로 'iced cafe'를 포함하고 있는 2개의 결과만 Hit된 것을 볼 수 있다.
### match_phrase
원하는 검색어의 공백을 포함해 정확히 일치하는 내용 검색이 가능하다.
```js
GET cafe/_search
{
  "query": {
    "match_phrase": {
      "title": "iced americano"
    }
  }
}
```
> 실행결과
```json
"hits" : {
  "total" : {
    "value" : 1,
    "relation" : "eq"
  },
  "max_score" : 1.2413131,
  "hits" : [
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "KwlS_X8BQ3yq82ANLq4s",
      "_score" : 1.2413131,
      "_source" : {
        "title" : "iced americano",
        "price" : "1500"
      }
    }
  ]
}
```
`slop`을 이용해 해당 검색 키워드 공백 사이 `slop`값 만큼 끼어드는 것을 허용할 수 있다.
```js
GET cafe/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "iced latte",
        "slop": 1
      }
    }
  }
}
```
> 실행결과
```json
"hits" : {
  "total" : {
    "value" : 1,
    "relation" : "eq"
  },
  "max_score" : 0.6763016,
  "hits" : [
    {
      "_index" : "cafe",
      "_type" : "coffe",
      "_id" : "LAlS_X8BQ3yq82ANoq7k",
      "_score" : 0.6763016,
      "_source" : {
        "title" : "iced cafe latte",
        "price" : "2500"
      }
    }
  ]
}
```
'iced'와 'latte' 사이에 'cafe'가 없지만 slop값만큼 단어가 끼어드는 것이 허용됨을 확인 가능

## Term Query

## 다중 인덱스 검색
여러 인덱스를 동시에 찾을 때 인덱스명을 `indexName1, indexName2`로 구분 할 수 있다.
```js
GET logkafka-2022.05.16,logkafka-2022.05.17,logkafka-2022.05.23/_search
{
  "from": 0, 
  "size": 10000,
  "query": {
    "match_all": {}
  }
}
```
같은 형태의 인덱스명을 동시에 검색할 경우에는(ex. `log-yyyy-MM-dd` 형태의 인덱스명) 아래와 같이 검색 가능
```js
GET logkafka*/_search
{
  "from": 0, 
  "size": 10000,
  "query": {
    "match_all": {}
  }
}
```

