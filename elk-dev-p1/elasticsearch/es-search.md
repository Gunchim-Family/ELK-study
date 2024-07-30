# 엘라스틱 서치 검색

## 1. 쿼리 컨택스트와 필터 컨택스트

### 쿼리 컨텍스트
- **역할**: 도큐먼트의 유사도(연관성)를 계산하여 정확한 검색 결과를 반환
- **스코어 계산**: 쿼리와 도큐먼트의 유사도를 기준으로 스코어를 매겨 순위가 높은 도큐먼트를 반환
- **예시**:
  ``` bash 
  GET kibana_sample_data_ecommerce/_search
  {
    "query": {
      "match": {
        "category": "clothing"
      }
    }
  }
  ```
  
### 필터 컨텍스트
- **역할**: 도큐먼트를 참/거짓으로 매칭하여 결과를 반환. 스코어 계산을 하지 않음
- **캐싱**: 필터는 캐싱되어 검색 성능을 향상시킬 수 있음
- **예시**:
    ``` bash
   GET kibana_sample_data_ecommerce/_search
    {
      "query": {
        "bool": {
          "filter": {
            "term": {
              "day_of_week": "Friday"
            }
          }
        }
      }
    }
    ```

## 2. 쿼리 스트링과 쿼리 DSL

### 쿼리 스트링
- **정의**: 간단한 쿼리를 한 줄로 표현. HTTP API 형식에서 사용되며, 필드와 값을 :로 구분.
- **예시**:
  ``` bash
  GET kibana_sample_data_ecommerce/_search?q=category:clothing
  ```

### 쿼리 DSL
- **정의**: 복잡한 쿼리를 JSON 형식으로 표현. 복잡한 검색 조건을 명확하고 구조적으로 표현 가능.
- **예시**:
  ``` bash
  GET kibana_sample_data_ecommerce/_search
  {
    "query": {
      "match": {
        "category": "clothing"
      }
    }
  }
  ```

## 3. 유사도 스코어
- **정의**: 질의문과 도큐먼트의 유사도를 표현하는 값. 스코어가 높을수록 찾고자 하는 도큐먼트에 가깝다.
- **계산 방식**: 문서 빈도(IDF)와 용어 빈도(TF)를 사용하여 스코어를 계산한다. 알고리즘은 BM25를 사용

```bash
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": "mary"
    }
  
  }
  , "explain": true
}
```

- **문서빈도(IDF)**: 특정 용어가 얼마나 자주 등장했는지 의미하는 지표
  - 전체 문서에서 자주 발생하는 단어일수록 가중치를 낮춘다 (중요하지않은 단어로 인식)
- **용어빈도(TF)**: 특정 용어가 하나의 도큐먼트에 얼마나 많이 등장했는지 의미하는 지표
  - 특정 용어가 도큐먼트에서 많이 반복되었다면 그 용어는 도큐먼트의 주제와 연관되어있을 확률이 높다
  - 하나의 도큐먼트에서 특정 용어가 많이 나오면 중요한 용어로 인식하고 가중치를 높인다

## 4. 쿼리

### 전문 쿼리와 용어 수준 쿼리
- **전문 쿼리**: 전체 텍스트 중에서 특정 용어나 용어들을 검색할 때 사용 
  - 구글/네이버 검색과 유사
  - Match, Match Phrase, Multi Match 등
- **용어 수준 쿼리**: 정확히 일치하는 용어 키워드 타입 
  - RDB의 WHERE 절과 유사
  - Term, Terms, Range 등

### match 쿼리
- **정의**: 전문 쿼리 중 가장 기본으로, 전체 텍스트 중에서 특정 용어나 용어들을 검색할 때 사용.
- **사용법**: OR, AND와 같은 논리 연산자를 사용하여 여러 조건을 결합.

```bash
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "match": {
      "customer_full_name": "Mary"
    }
  }
}
```

### match_phrase 쿼리
- **정의**: 구(Phrase) 검색을 할 때 사용. 용어가 모두 포함되면서 순서도 동일해야 함.

```bash
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "match_phrase": {
      "customer_full_name": "Mary baily"
    }
  }
}
```

### 멀티 매치 쿼리
- **정의**: 여러 필드에 대해 특정 용어를 검색하고, 각 필드에서의 개별 스코어 중 가장 큰 값을 대표 스코어로 사용.

```bash
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "multi_match": {
      "query": "Mary",
      "fields": [
        "customer_first_name", 
        "customer_full_name",
        "customer_last_name"
      ]
    }
  }
}
```

### term 쿼리
- **정의**: 정확히 일치하는 용어를 검색

```bash
# RDB의 WHERE 절과 유사하며 Mary baily라는 이름을 가진 도큐먼트를 검색
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["customer_full_name"],
  "query": {
    "term": {
      "customer_full_name": "Mary baily"
    }
  }
}
```

### 범위 쿼리
- **정의**: 특정 범위 내의 값을 검색 주로 날짜나 숫자 범위를 지정할 때 사용
- **형식** : 날짜 형식: "YYYY-MM-DD"
```bash
# gte: 이상, lte: 이하
GET kibana_sample_data_ecommerce/_search
{
 "query": {
   "range": {
    "day_of_week": {
      "gte": 10,
      "lte": 20
    }
   }
 }
}
```

### 논리 쿼리
- **정의**: 여러 쿼리를 결합하여 복합적인 조건을 표현.
- **논리 쿼리 타입**:
  - **must**: 참인 도큐먼트 (복수 쿼리 : AND 조건)
  - **must_not**: 거짓인 도큐먼트
  - **should**: 단독으로 참인 도큐먼트 (복수 쿼리 : OR 조건)
  - **filter**: 예/아니오 필터 컨텍스트 수행

### 예시
- **must 쿼리**:

```bash
GET kibana_sample_data_ecommerce/_search
{
 "query": {
    "bool": {
      "must": {
        "match": {
          "customer_full_name": "mary"
        }
      }
    }
   }
}
```

- **복수 must 쿼리**:

```bash
GET kibana_sample_data_ecommerce/_search
{
  "_source": ["day_of_week","customer_full_name"], 
 "query": {
    "bool": {
      "must": [
        { "term": { "day_of_week": "Sunday" } },
        { "match": { "customer_full_name": "mary" } }
      ]
    }
   }
}
```

## 패턴을 이용한 쿼리

### 와일드카드 쿼리
- **정의**: 와일드카드 문자를 사용하여 패턴 매칭을 수행하여 용어 수준 쿼리임에 따라 SQL LIKE문처럼 특정 문자열 찾기용으로 부적합
- **문자**:
  - `*`: 공백 포함 모든 문자열
  - `?`: 오직 한 가지 문자 매칭

### 정규식 쿼리
- **정의**: 정규 표현식을 사용하여 패턴 매칭을 수행
