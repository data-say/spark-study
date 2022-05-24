## MapReduce

- HDFS에서 데이터를 읽어 연산 처리 후 그 결과를 다시 HDFS에 기록
- `Map`, `Reduce` 작업 반복
- 처리 결과를 HDFS에 `write` 할 때 데이터는 복제되고 직렬화되며 기본적인 Disk IO가 발생하기 때문에 다소 느린편임
- 아래 작업을 하는데에 있어 비효율적
    - Iterative algorithms (Machine Learning, Graph & Network Analysis)
    - Interactive Data Mining (R, Excel, Ad-hoc Reporting, Searching)
    

## Spark

- 반복적으로 사용할 데이터를 메모리에 cache하여 Disk 보다 빠르게 재사용 가능
- 메모리에 데이터를 로딩하여 처리할 경우 디스크에 있는 데이터를 처리하는 것보다 10 ~ 100배 가량 빠름
- `Map`, `Reduce` 2개의 Stage로 얽매이지 않음
    - 다양한 상위레벨 api를 체인과 같이 연속적으로 사용하여 원하는 만큼 데이터 연산 처리 가능 (flatMap > filter > map > reduceByKey > …)
    - 메모리에 cache (cache, persist)
    - 반복연산 알고리즘 실행 및 대화형 질의 반복 수행하므로서 보다 향상된 성능 보여줌
    

## Spark vs Hadoop - Speed

- **데이터 처리 성능 비교**
    - 2014년 100TB의 데이터를 정렬하는 Daytona Gray Sort Benchmark에서 우승
        - 2013년에는 Hadoop MapReduce가 우승
    - Hadoop MapReduce 대비 100TB의 데이터에 대해 약 10배 더 적은 서버 리소스로 3배 더 빠른속도의 성능을 보임
        - 메모리 cache를 사용하지 않고 디스크 기반으로 했을 때
- **클라우드 인프라에 대한 비용 측면 처리 성능 비교**
    - Spark은 100TB 데이터 정렬하는데 사용되는 비용에 대한 Benchmark 인 2016 CloudSort Benchmark에서 우승
        - TB 당 $1.44 (TB 당 $4.51 - 2014 보다 3배 더 비용 효율적임)
- **머신러닝 알고리즘 훈련 시간 측면 처리 성능 비교**
    - Hadoop MapReduce: 매 반복마다 110초 소요
    - Spark: 첫 반복을 통해 데이터를 읽어 메모리에 cache 하는데 80초 소요, 이후에는 cache된 데이터를 사용하므로 1초 소요
    

## Spark vs Hadoop - Ease of Use

- **Spark의 경우 Hadoop MapReduce보다 2~5배 간결한 코드로 Word Count 가능**
    - Spark이 제공하는 다양한 api를 이용해 함수형 방식으로 코드 작성 가능
    - 필요할 경우 다양한 상위 레벨 api를 체인과 같이 연속적으로 사용하여 원하는 만큼 데이터 연산 반복 수행 가능
- **Low Level API vs. High Level API**
    - Hadoop MapReduce
        - map, reduce
    - Spark
        - map, reduce, sample, filter, count, take, groupBy, fold, first, sort, reduceByKey, partitionBy, union, groupByKey, mapWith, join, cogroup, pipe, leftOuterJoin, cross, save, rightOuterJoin, zip 등 80여개 이상의 api 제공
