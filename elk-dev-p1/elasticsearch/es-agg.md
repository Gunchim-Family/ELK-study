# 엘라스틱 서치 : 집계

### 요약
기본적으로 버킷 집계, 매트릭 집계가 있음
- 버킷 집계는 특정 기준에 맞춰 도큐먼트를 그룹핑하는 역할
- 매트릭 집계는 특정 필드를 기준으로 통계 값을 계산하는 역할

위 두개의 집계를 조합해서 통계를 계산할 수 있으며 집계의 결과를 다음 단계에 이용하는 파이프라인 집계도 있음

## 집계 (Aggregation) 개요
집계(Aggregation)는 데이터를 그룹핑하고 통계 값을 얻는 기능으로, SQL의 `GROUP BY`와 유사합니다. 
집계는 여러 단계로 구성되며, 각 단계는 버킷(Bucket)과 메트릭(Metric)으로 구분됩니다. 
키바나(Kibana)의 데이터 시각화와 대시보드는 대부분 집계 기능을 기반으로 동작합니다.

### 집계 요청-응답 형태

```bash
GET <index>/_search
{
  "aggs": {
    "AGG_NAME": {
      "AGG_TYPE": {}
    }
  }
}
```

## 집계 종류
집계는 크게 매트릭 집계와 버킷 집계로 나뉩니다.

### 매트릭 집계
- 특정 필드를 기준으로 통계 값을 계산하는 것이 목적입니다. 
- 필드의 최소/최대/합계/평균/중간값 같은 통계 결과를 보여주며, 필드 타입에 따라서 사용 가능한 집계 타입에 제한이 있습니다.

#### 매트릭 집계 종류
- **avg**: 필드의 평균값을 계산합니다.
- **stats**: 필드의 최소값(min), 최대값(max), 합계(sum), 평균값(avg), 도큐먼트 개수(count)를 한 번에 볼 수 있습니다.
- **cardinality**: 필드의 유니크한 값 개수를 보여줍니다.
- **geo-centroid**: 필드 내부의 위치 정보의 중심점을 계산합니다.

#### 예시: 평균값 구하기 (avg)

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      "avg": {
        "field": "products.base_price"
      }
    }
  }
}
```

#### 예시: 백분위 구하기 (percentiles)

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_aggs": {
      "percentiles": {
        "field": "products.base_price",
        "percents": [25, 50, 99]
      }
    }
  }
}
```

#### 예시: 필드의 유니크한 값 개수 확인 (cardinality)

```bash
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

#### 응용: 검색 결과를 활용한 집계

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "term": {
      "day_of_week": "Monday"
    }
  },
  "aggs": {
    "query_aggs": {
      "sum": {
        "field": "products.base_price"
      }
    }
  }
}
```

### 버킷 집계
- 버킷 집계는 특정 기준에 맞춰서 도큐먼트를 그룹핑하는 역할을 합니다. 
- 버킷은 도큐먼트가 분할되는 단위로 나뉜 각 그룹을 의미합니다.

#### 버킷 집계 종류

- **histogram**: 숫자 타입 필드를 일정 간격으로 분류합니다.
- **range**: 필드를 사용자가 지정하는 범위 간격으로 분류합니다.
- **terms**: 필드에 많이 나타나는 용어(값)들을 기준으로 분류합니다.
- **filter**, **significant_terms** 등 다양한 버킷 집계가 존재합니다.

#### 예시: 숫자 타입 필드를 일정 간격으로 구분 (histogram)

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "histogram_agg": {
      "histogram": {
        "field": "products.base_price",
        "interval": 100
      }
    }
  }
}
```

#### 예시: 범위 집계 (range)

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "range_aggs": {
      "range": {
        "field": "products.base_price",
        "ranges": [
          {"from": 0, "to": 50},
          {"from": 50, "to": 100},
          {"from": 100, "to": 200}
        ]
      }
    }
  }
}
```

#### 예시: 필드의 유니크한 값을 기준으로 버킷 나누기 (terms)

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_agg": {
      "terms": {
        "field": "day_of_week",
        "size": 5,
        "show_term_doc_count_error": true,
        "shard_size": 100
      }
    }
  }
}
```

### 집계의 조합

매트릭 집계와 버킷 집계를 조합해서 통계를 계산할 수 있습니다.

- 버킷 집계 후 매트릭 집계 사용하기
- 서브 버킷 집계 : 버킷 집계 > 버킷 집계

#### 예시: 버킷 집계 후 매트릭 집계 사용하기

```bash
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "term_agg": {
      "terms": {
        "field": "day_of_week",
        "size": 5,
        "show_term_doc_count_error": true,
        "shard_size": 10
      },
      "aggs": {
        "avg_aggs": {
          "avg": {
            "field": "products.base_price"
          }
        }
      }
    }
  }
}
```

### 파이프라인 집계
- 이전 집계 결과를 다음 단계에 이용하는 집계입니다.
- **부모 집계**: 기존 집계 결과를 참고해 새로운 집계 결과를 생성 (기존 집계 내부에서 나옴).
- **형제 집계**: 기존 집계를 참고해 집계 (기존 집계와 동일한 라인).
