## 엘라스틱서치 집계

* 엘라스틱 서치에서 집계를 통해 통계, 비율 계산과 같은 결과를 얻을 수 있다.
* 집계는 search API 요청 본문에 aggs 를 주어 사용할 수 있고, 이를 통해 쿼리 결과에 대한 집계를 얻을 수 있다.
* 집계는 통계나 계산에 사용되는 `메트릭 집계`와 도큐먼트 그루핑에 사용되는 `버킷 집계`가 있다.

### 메트릭 집계

* 필드의 최소/최대/합계/평균/중값 처럼 통계 결과를 얻을 때 사용하는 집계
* 종류
    * avg : 필드의 평균값
    * min : 필드의 최솟값
    * max : 필드의 최댓값
    * sum : 필드의 총합
    * percentiles : 필드의 백분위 값 계산
    * stats: 필드의 min, max, sum, avg, count(도큐먼트 개수)를 한번에 조회한다.
    * cardinality : 필드의 유니크한 값 개수
    * geo-centroid : 필드 내부의 위치 정보의 중심점 계산

#### 기본 사용법 - 필드 합 계산

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      // 집계의 이름을 stats_aggs로 셋팅한다.
      "avg": {
        // products.base_price의 평균을 구한다.
        "field": "products.base_price"
      }
    }
  }
}
```

#### 백분위 계산 집계

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      "percentiles": {
        "field": "products.base_price",
        "percents": [
          // 25%, 50%에 속하는 데이터를 조회한다. 중간값은 50이고 최댓값은 100이다.
          25,
          50
        ]
      }
    }
  }
}
```

#### cardinality 계산

```bash
// precision_threshold 기본값 3000, 최대 40000까지 설정 가능
// 해당값이 크면 리소스 소모가 큰 대신 정확도가 올라간다.
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      "cardinality": {
        "field": "day_of_week",
        "precision_threshold": 100
      }
    }
  }
}
```

#### 결과 내 집계

```bash
//  "day_of_week": "Monday" 로 필터링 후, 집계를 진행한다.
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "term": {
      "day_of_week": "Monday"
    }
  },
  "aggs": {
    "stats_aggs": {
      "sum": {
        "field": "products.base_price"
      }
    }
  }
}
```

### 버킷 집계

* 특정 기준에 따라 도큐먼트를 그루핑할 때 사용하는 집계
* 종류
    * histogram : 숫자 타입 필드 일정 간격으로 분류
    * date_histogram : 날짜/시간 타입 필드 일정 기간으로 분류
    * range : 숫자 타입 필드 사용자 지정 범위 간격으로 분류
    * date_range : 날짜/시간 타입 필드 사용자 지정 범위 간격으로 분류
    * terms : 필드에 많이 나타나는 용어들을 기준으로 분류
    * significant_terms : terms 버킷과 유사하지만 현재 검색 조건에서 유의미한 값들을 기준으로 분류한다.
    * filters : 그룹에 포함시킬 조건을 직접 지정한다.

#### histogram 집계
* 아래 쿼리는 필드값 100을 간격으로 도큐먼트 개수를 집계한다.
```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "histogram_aggs": {
      "histogram": {
        "field": "products.base_price",
        "interval": 100
      }
    }
  }
}
```

#### range 집계

* 범위에 따라 집계 결과가 달라질 수 있다.
    * 만약 도큐먼트에서 필드 a가 [10,30,90] 이라는 값을 가지는 경우, 범위를 0~100으로 잡으면 결과가 1이지만 범위를 0~10으로 잡으면 결과는 3이다.
* 각 범위에 따른 도큐먼트 개수를 결과로 얻을 수 있다.
* 아래 쿼리는, products.base_price를 기준으로 4개의 번위로 집계한다. 
```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "range_aggs": {
      "range": {
        "field": "products.base_price",
        "ranges": [
          {
            "from": 0,
            "to": "50"
          },
          {
            "from": 50,
            "to": "100"
          },
          {
            "from": 100,
            "to": "200"
          },
          {
            "from": 200,
            "to": "1000"
          }
        ]
      }
    }
  }
}
```

#### terms 집계

* "size"는 지정하지 않으면 기본 10이다.
* 아래 쿼리의 경우 day_of_week 기준으로 도큐먼트 개수가 많은 상위 6개 버킷을 조회한다.
* 데이터를 여러 노드에 분산하여 저장한 경우, 데이터 집계 결과가 정확하지 않을 수 있다.
    * 정확하지 않은 결과가 나오는 경우, 결과의 `doc_count_error_bound`를 참고하자
    * 파라미터에 `show_term_doc_count_error`를 주어 검색 후 이상이 있는 경우, `shard_size`를 지정해주어 정확도를 높일 수 있다.
* 샤드의 수는 `size(버킷 수) *1.5 +10` 으로 계산하면 된다.

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "range_aggs": {
      "terms": {
        "field": "day_of_week",
        "size": 6
      }
    }
  }
}
```

#### 서브 버킷 집계

* 버킷 안에서 다시 버킷 집계를 요청하는 집계
* 아래 쿼리를 보면 `histogram`을 기준으로 100단위 버킷을 나눈 후, 각 버킷 내부에 terms 기준으로 버킷을 나눈다. 
* 성능 등의 이유로 서브 버킷은 2단계를 초과해 사용하지 말자
```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "histogram_aggs": {
      "histogram": {
        "field": "products.base_price",
        "interval": 100
      },
      "aggs": {
        "term_aggs": {
          "terms": {
            "field": "day_of_week",
            "size": 2
          }
        }
      }
    }
  }
}
```

#### 파이프라인 집계 
* 이전 결과를 다음 단계에서 사용하는 집계 
* 부모 집계와 형제 집계라는 개념이 있다. 
  * 부모 집계 : 기존 집계 결과를 이용해 새로운 집계 생성
  * 형제 집계 : 기존 집계 참고하여 집계 수행, 기존 집계와 동일 선상 

