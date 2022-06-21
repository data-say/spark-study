## Spark Programming Model - SparkContext

- Spark Shell과 같은 Interactive Shell은 내부적으로 알아서 SparkContext를 생성해주기도 함
- Spark 프로그램은 **SparkContext**로부터 시작됨
    - Cluster Manager에게 Executor 생성 요청
    - Executor의 task에서 실제 데이터를 읽고 처리하고 저장하는 작업 진행
    - Data Cache도 Executor의 메모리 사용

## Spark Programming Model - RDD

- SparkContext를 통해 대용량 데이터를 담는 그릇인 RDD 생성
- RDD는 드라이브에 위치하지 않고 partition 단위로 분리되어 다수의 Executor에 분산되어 존재
- RDD가 제공하는 api를 통해 RDD에 담겨진 데이터에 대한 다양한 변경작업을 함수형 코드로 작성
- RDD에 담겨진 데이터 처리는 하나의 파티션에 대해 하나의 스레드가 동작함

```scala
sc = SparkContext()
rdd = sc.textFile("hdfs://...")
rdd2 = rdd.filter(...)
rdd3 = rdd2.repartition(100) # 분산병렬화 수준 파라미터
rdd4 = rdd3.flatMap(...)
rdd5 = rdd4.map(...)
rdd5.cache
...
```

## Spark Programming Model - RDD APIs

1. **Transformation API**: RDD 데이터 변경 api
    - 설명
        - 변경된 데이터를 담고 있는 새로운 rdd 데이터 리턴하는 api (기존 rdd 데이터 직접 변경 X)
        - RDD 데이터는 변경할 수 없는 immutable한 데이터 구조를 가짐
        - 데이터 처리 작업 코드는 Driver가 아닌 Executor에 전달되어 분산병렬로 실행됨
    - 특징
        - Lazy하여 즉시 실행되지 않고 Action API가 호출될 때까지 미뤄두고 Action API가 호출될 때 그제서야 작업 순차 실행
            - 실행 최적화를 위한 방법
    - 예시
        - `map(func)`
        - `filter(func)`
        - `distinct()`
        - 데이터 변경 로직을 함수 형태로 전달받음
2. **Action API**: RDD 데이터를 Driver로 가져오거나 외부 저장소에 저장하기 위한 api
    - 설명
        - 실행될 때마다 외부에서 데이터를 새로 읽어와 최초의 RDD부터 새로 생성하여 순차적으로 작업 처리
        - 변경된 RDD 데이터 결과에 대한 최종 처리 작업 담당
            - 미뤄두었던 transformation api 처리 작업 진행 후 호출된 action api에 따라 RDD 내용 연산하여 driver나 외부 저장소에 저장
    - 예시
        - `count()`
        - `reduce(func)`
        - `collect()`
        - `take(n)`
        - `saveAsTextFile(path)`
3. **Persistence API**: 반복적으로 자주 사용되는 RDD는 성능향상을 위해 메모리나 디스크에 cache 하기 위한 api
    - 설명
        - **cahce된 RDD**에 대해 Action API를 호출하면 cache된 RDD에서부터 작업 처리 (외부에서 데이터를 새로 읽어와 최초의 rdd로부터 다시 생성 X) → **처리 속도 측면 성능 향상**
    - 예시
        - `persist(StorageLevel)`
        - `cache()`

## Example

```bash
## pyspark interactive shell 실행
YARN_CONF_DIR=/user/spark3/conf2 ./bin/pyspark --master yarn --num-executors 5 --executor-cores 2 --executor-memory 2G

## SparkContext 확인
sc 
# <SparkContext master=yarn appName=PySparkShell>

## 기존 SparkContext 중지
sc.stop()
# driver는 있으나 executor program은 모두 사라짐
# 중지하면 할당받은 executor를 cluster에 다시 반환

## SparkContext 새로 생성
sc = SparkContext()
# executor process가 새로 실행됨
# SparkContext가 생성되면서 Hadoop YARN Cluster Manager에게 executor 생성을 요청하여 새로 할당받음

## RDD 생성
rdd = sc.textFile("hdfs://spark-master-01:9000/user/data/airline_on_time")

## RDD partition 확인
rdd.getNumPartitions() 
# 55

## RDD filter api (transformation api)를 통해 새로운 RDD 생성
rdd2 = rdd.filter(lambda line: line.startswith("2008"))

## RDD count api (action api)
rdd2.count()
# 2389217
# driver web ui를 통해 소요 시간 등 확인 가능
# > 55개의 task가 실행됨, 49초 소요

## RDD cache (persistence api)
rdd2.cache()
# 실제 cache 수행을 위해서는 action 수행 필요

## action api 호출
rdd2.count()
# 2389217
# driver web ui `Storage 탭`에서 cache 확인 가능

## 연산 속도 재확인
rdd2.count()
# 속도 빨라짐, 1초 소요

## cache 해제
rdd2.unpersist()

## action api 호출
rdd2.count()
# 속도 느려짐, 41초 소요

## Partition 개수 조정
rdd3 = rdd2.repartition(6)

## Partition 개수 확인
rdd3.getNumPartitions()
# 6

## action api 호출
rdd3.count()
# 48초 소요

## action api 호출
rdd3.count()
# 속도 빨라짐, 1초 소요
# 동일한 결과를 뱉어낼 첫번째 stage를 스킵하고 첫번째 stage의 연산 결과인 shuffle 데이터를 읽어 두번째 stage를 바로 실행
# 속도 빨라지는 효과
```