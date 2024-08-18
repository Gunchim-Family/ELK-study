# Beats

### 요약
비츠(Beats)는 Logstash와 달리 하나의 출력(output)만을 가질 수 있는 가볍고 사용하기 쉬운 데이터 수집기입니다.

### 소개
- Beats는 다양한 시스템의 이벤트를 수집하여 Logstash나 Elasticsearch로 전송하는 가볍고 간편한 데이터 수집 도구입니다. 
- 설정이 간편하며, 별도의 데이터 가공을 위한 프로그래밍 작업이 필요하지 않습니다. 
- 다양한 비트(Beats)들이 존재하며, 각각 특정 목적에 최적화되어 있습니다.
- **주요 비트(Beats) 종류**:
  - **파일비트(Filebeat)**: 로그 파일을 실시간으로 수집
  - **메트릭비트(Metricbeat)**: 시스템 및 서비스의 메트릭 수집
  - **패킷비트(Packetbeat)**: 네트워크 데이터 수집

### Logstash와의 비교

- **Logstash**:
  - 다양한 플러그인을 지원하여 범용성이 높지만, 상대적으로 무겁습니다.
  - 대규모 이벤트 가공에 적합하며, 복잡한 데이터 처리 작업을 수행할 수 있습니다.

- **Beats**:
  - 범용성을 포기하고 특정 목적의 데이터 수집에 최적화되어 있어 가볍고 빠릅니다.
  - 애플리케이션 성능에 미치는 영향을 최소화하면서 필요한 이벤트를 수집합니다.
  - 기본적인 이벤트 가공을 일부 지원합니다.

### Beats의 활용

- **데이터 흐름**:
  - Beats에서 수집한 데이터 → Elasticsearch로 전송
  - Beats에서 수집한 데이터 → Logstash로 전송 → Elasticsearch로 전송

### 파일비트(Filebeat) 아키텍처

Filebeat는 로그 파일을 실시간으로 읽어들이며, 구성 요소는 다음과 같습니다:

- **입력(Input)**: 수집할 파일의 경로를 설정합니다.
- **하베스터(Harvester)**: 지정된 파일을 읽고 데이터를 수집합니다. 파일이 변경될 때마다 데이터를 지속적으로 읽습니다.
- **스펄러(Spooler)**: 수집한 데이터를 Elasticsearch나 Logstash로 전송합니다.

Filebeat는 파일의 새로운 내용을 모니터링하고, 새로운 파일이 발견되면 하베스터를 생성하여 데이터를 읽어들입니다.

### 입력(Input) 타입

- **log**: 일반적인 로그 파일 수집
- **container**: 컨테이너 로그 수집
- **s3**: AWS S3에서 로그 수집
- **kafka**: Kafka에서 데이터 수집

### 출력(Output) 타입

- **elasticsearch**: 데이터를 Elasticsearch로 전송
- **logstash**: 데이터를 Logstash로 전송
- **kafka**: 데이터를 Kafka로 전송
- **console**: 데이터를 콘솔에 출력

### filebeat.yml 설정

- **ignore_older**: 설정한 시간 이전의 로그는 무시

```yaml
ignore_older: 2h
```

- **간단한 정제 작업**: 포함/제외할 로그 패턴 설정

```yaml
include_lines: ['^ERR', '^WARN']
exclude_lines: ['^DBG']
exclude_files: ['\.gz$']
```

- **멀티라인 처리**: 여러 줄에 걸친 로그를 하나로 처리

```yaml
multiline.pattern: '^[[:space:]]'  
multiline.negate: false
multiline.match: after
```

### 모듈

Beats는 다양한 모듈을 통해 특정 데이터 소스를 쉽게 수집할 수 있습니다:

- **aws**: AWS 서비스 로그 수집
- **cef**: Common Event Format 로그 수집
- **elasticsearch**: Elasticsearch 로그 수집
- **logstash**: Logstash 로그 수집
