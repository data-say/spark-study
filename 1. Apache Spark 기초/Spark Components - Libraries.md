## MapReduce의 한계

- 프로그래밍이 어려움 (Java - Map, Reduce만으로 개발)
    - 대화형 개발환경 X, complie
- 배치 기반 환경 - 성능 병목 현상 발생
- 특정 use case에는 적합하지 않음
    - 배치 이외의 프로세싱 방식의 대규모 application에는 적합하지 않음
- 다양한 특화 시스템 제공
    - Pregel, Giraph, Dremel, Drill, Tez, Impala, GraphLab, Storm, S4

## Spark의 접근

- 별도의 다양한 특화 시스템 대신 MapReduce를 일반화하여 동일 엔진 내에서 다양한 application 지원
- 아래 2가지 컨셉 추가
    - Fast data sharing
    - General task DAGs
- 동일 엔진에서 여러 처리를 지원함
    - 엔진 효율적으로 동작
    - 엔진을 더 심플하게 사용 가능
- 유저 입장
    - 반복적인 read, write 작업
        - HDFS read > ETL > HDFS write / HDFS read > train > HDFS write / HDFS read > query > HDFS write
    - 하나의 데이터로 바로 학습 및 추론 진행 가능
        - HDFS read > ETL > train > query

## Example

- Spark SQL + MLlib + Spark Streaming

```scala
// Spark SQL
df.createOrReplaceTempView("historic_tweets")
val points = spark.sql("select latitude, longitude from historic_tweets")

// MLlib
val kmeans = new KMeans().setK(10).setSeed(1L)
val model = kmeans.fit(points)

// Spark Streaming
TwitterUtils.createStream(ssc, None)
						.map(t => (model.predict(t.location), 1))
						.reduceByKeyAndWindow(_+_, Seconds(60))
```

## Spark Components

- 하나의 프로그램에서 다양한 processing type들을 라이브러리 형태로 결합하여 사용 가능
- 코드 재사용성이 높음

## Spark Components - Spark SQL

- 대용량 정형 데이터에 대한 쿼리 기능 제공 (비정형, 반정형도 가능)
- 다양한 포맷의 파일, 관계형 데이터베이스, 데이터웨어하우스의 데이터도 사용 가능
- JDBC, ODBC 연결 가능
- Hive Query 수정없이 사용 가능
- 표준 ANSI-SQL 사용

## Spark Components - Spark Streaming

- 대용량 스트리밍 데이터에 대한 실시간 분석을 위한 라이브러리
- Kafka, Flume 등의 외부 스트림 소스로부터 대용량 스트림 데이터를 지속적으로 읽어와 분산/병렬로 분석/처리 후 다시 외부로 결과를 전달
- RDD 기반의 Spark Core 엔진 사용
    - 작은 단위의 micro-batch기반의 스트림 처리 진행
        - 초/분 단위 스트림 데이터를 RDD 기반 배치 처리 후 해당 배치의 RDD를 순차 처리
- 시간의 흐름에 따른 지속적인 상태 관리 가능
    - stateful, joins, aggregates, window 등
- Fault Tolerant
- Structured Streaming
    - Catalyst Optimizer
    - 지속적으로 들어오는 스트림 데이터를 unbounded table에 논리적으로 무한대로 추가하는 컨셉

## Spark Components - Spark MLlib

- 확장가능한 기계학습 라이브러리
- 빠른 반복 연산 가능
- python, R의 다양한 라이브러리 포함
- 다양한 알고리즘
    - 분류, 예측, 추천
    - 클러스터링
    - 모델 평가, 하이퍼파라미터 튜닝
    - ML 파이프라인 구축

## Spark Components - GraphX

- RDD는 edge와 vertex를 테이블 형태로 가짐
- 테이블과 그래프로 구성가능한 통합된 뷰 제공
- 대용량 그래프 데이터를 효율적으로 분산 처리하기 위한 Pregel API 제공
- PageRank, Label Propagation, SVD++, Strongly Connected Components, Triangle Count, Shortest Paths

## Spark Third-Party 라이브러리

- spark-als
- spark-csv
- tensorframes