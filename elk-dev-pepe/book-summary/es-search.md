## 엘라스틱서치 - 검색

* 쿼리 컨텍스트 : 연관성에 따라 스코어 결과를 제공한다. 유사도 알고리즘에 의해 가장 유사한 도큐먼트가 상단에 위치한다.
* 필터 컨텍스트 : 스코어는 사용하지 않고, '예/아니요'의 결과를 제공한다. 스코어를 계산하지 않기 때문에 성능이 더 좋다.

쿼리 컨텍스트
```
GET kibana_sample_data_ecommerce/_search
{
  "query":{
    "match":{
      "category":"clothing"
    }
  }
}
```

필터 컨텍스트
```
GET kibana_sample_data_ecommerce/_search 
{
  "query":{
    "bool":{
      "filter":{
        "term":{
          "day_of_week": "Friday"
        }
      }
    }
  }
}
```


### 쿼리 스트링 & 쿼리 DSL
쿼리 스트링 : WEB에서 사용하는 쿼리 스트링의 방식과 유사하다. 간단한 검색시 유용하다.
```
GET kibana_sample_data_ecommerce/_search?q=customer_full_name:Mary
```

쿼리 DSL : REST API 요청시 BODY안에 JSON형태로 조건을 작성하여 요청하는 방식
```
GET kibana_sample_data_ecommerce/_search 
{
  "query":{
    "match":{
      "customer_first_name": "Mary"
    }
  }
}
```


### 유사도 스코어

#### BM25 알고리즘
* TF(Term Frequency), IDF(Inverse Document Frequency) 개념에 문서 길이를 고려한 알고리즘
* 검색어가 문서에서 얼마나 자주 나타나고, 중요한지 등을 판단한다.

* IDF(Inverse Document Frequency)
    * DF는 특정 용어가 얼마나 자주 등장했는지를 의미한다.
    * 'to', 'the', '그리고', '그러나'와 같은 접속 부사의 경우 어떤 문서든 자주 등장한다. 자주 등장하는 단어일 수록 중요도가 낮아진다.
    * ES에서는 도큐먼트 내 빈도가 적을 수록 높은 가중치를 주는 IDF(문서 빈도의 역수)라는 개념을 도입하였다.
* TF(Term Frequency)
    * 특정 용어가 하나의 도큐먼트에서 얼마나 자주 등장했는지를 의미한다.
    * TF가 높다는 것은 해당 용어가 도큐먼트의 주제일 확률이 높음을 의미한다.
* 최종 스코어의 경우 IDF*TF*boost(고정값 2.2) 연산의 결과이다.


### 쿼리
#### 전문 쿼리(match)와 용어 쿼리(term) 차이
* 전문 쿼리
    * 전문 쿼리는 텍스트 타입으로 매핑
    * 전문 쿼리 블로그처럼 텍스트가 많은 필드에서 특정 용어 검색할 때 사용한다.
    * 검색어도 토큰화한다. 일반적인 분석기의 경우 대문자를 소문자로 변환하기 때문에 대소문자 관계없이 검색이 가능하다.
    * 전문 쿼리의 종류에는 매치쿼리, 매치 프레이즈 쿼리, 멀티 매치 쿼리, 쿼리 스트링 쿼리가 있다.
* 용어 쿼리
    * 용어 수준 쿼리는 정확한 용어를 검색을 위해 키워드 타입으로 매핑해야 한다.
    * 키워드 타입의 경우 인덱싱 과정에서 분석기를 사용하지 않음
    * 정확한 용어를 검색할 때 사용한다.(대소문자를 구분함) 주로 숫자, 날짜, 범주형 데이터를 정확하게 검색할 때 사용하며 RDB의 WHERE절과 비슷한 역할을 한다.
    * 용어 수준 쿼리에는 용어 쿼리(term), 용어들 쿼리(terms), 퍼지 쿼리(fuzzy) 등이 있다.

#### 매치 쿼리
* 대표적인 전문 쿼리로 젙체 텍스트 중 특정 용어나 용어들을 검색 시 사용한다.
* 매치 쿼리를 사용하기 위해서는 인덱스의 필드명을 알아야한다.
* `GET kibana_sample_data_ecommerce/_mapping`와 같이 _mapping으로 인덱스에 포함된 필드를 확인할 수 있다.


```
GET kibana_sample_data_ecommerce/_search
{
  "_source":["customer_full_name"],
  "query":{
    "match":{
      "customer_full_name":"mary bailey"
    }
  }
}
```
위의 쿼리 조건 customer_full_name의 경우 `[marry, bailey]`으로 토큰화 된다.   
용어의 공백을 기준으로 OR로 인식하기 때문에 위의 쿼리는 marry or bailey의 조건으로 둘 중 하나라도 포함된 도큐먼트가 매칭된다.

```
GET kibana_sample_data_ecommerce/_search
{
  "_source":["customer_full_name"],
  "query":{
    "match":{
      "customer_full_name":{
        "query":"mary bailey",
        "operator": "and"
    }
  }
 }
}
```
operator and 옵션을 사용하여 두 토큰을 가진 도큐먼트를 조회할 수 있다. (디폴트 OR)

#### 매치 프레이즈 쿼리 
* 전문 쿼리의 한 종류로 구(phrase)를 검색할 때 사용한다. 
* 구는 동사가 아닌 2개 이상의 단어를 합성하여 만든 단어를 의미한다. 
* match와 유사하지만, 단어의 순서까지 체크한다. 
```
GET kibana_sample_data_ecommerce/_search
{
  "_source":["customer_full_name"],
  "query":{
    "match_phrase":{
      "customer_full_name":{
        "query":"mary bailey",
        "operator": "and"
    }
  }
 }
}
```

#### 용어 쿼리 
* 대표적인 용어 수준 쿼리로 검색어가 분석기에 의해 토큰화되지 않는다. 
* 따라서 대소문자까지 정확이 일치하는 도큐먼트와 매칭이 된다. 
* 현재 `customer_full_name`필드의 경우 텍스트로 매핑되어 있어 `[mary, bailey]`형태로 저장되어 있다. 
* 따라서 다음과 같이 요청을 해야한다. `"customer_full_name.keyword": "Mary Bailey"`
```
GET kibana_sample_data_ecommerce/_search
{
  "_source":["customer_full_name"],
  "query":{
    "term":{
      "customer_full_name.keyword": "Mary Bailey"
  }
 }
}
```

#### 논리 쿼리 (bool)
* 쿼리를 조합해서 사용하는 방식으로 복잡한 조건으로 검색할 때 사용한다.
* `must`, `must_not`, `should`, `filter`를 활용하여 전문 쿼리나 용어 쿼리, 범위 쿼리, 지역 쿼리 등을 조합해서 사용할 수 있다.
* must 
  * 조건이 반드시 일치해야 도큐먼트를 매칭한다. 
  * 복수 쿼리 실행시 AND연산 한다. 
* must_not 
  * 조건이 일치하지 않는 도큐먼트를 매칭한다. 
  * 다른 타입과 같이 사용시, 도큐먼트에서 제외한다. 
* should 
  * should타입 하나만 사용시 must와 같은 결과를 얻는다. 
  * 여러 개의 should 조건이 있을 때는 OR연산을 한다. 
  * 다른 타입과 사용하는 경우 검색 결과에 영향을 주지 않고, 스코어에만 활용된다. 
  * 도큐먼트의 검색 순위 최적화 등에 사용할 수 있다. 
* filter 
  * 기본적으로 must와 같은 동작을 한다.
  * must와 달리 필터 컨텍스트로 동작하기 때문에 스코어 계산을 하지 않는다. 
  * 즉, 성능 상의 이점이 있다.

```
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "products.category": "Women's Clothing"
          }
        },
        {
          "range": {
            "taxful_total_price": {
              "gt": 10
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "customer_gender": "MALE"
          }
        }
      ],
      "should": [
        {
          "match": {
            "customer_first_name": "Mary"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "currency": "EUR"
          }
        }
      ]
    }
  }
}
```
products.category는 여성복, taxful_total_price는 10 이상,customer_gender는 남성 제외 ,이름이 Mary인 경우 가중치, 환률은 EUR인 데이터를 조회하는 쿼리 예시


