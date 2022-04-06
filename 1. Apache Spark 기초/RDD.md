## RDD

`RDD (Resilient Distributed Dataset)` : Spark의 핵심 데이터 모델로 RDD는 실패로부터 쉽게 회복가능한 분산 데이터셋 구조를 추상화한 모델

- Resilient
    - 클러스터의 한 노드가 실패하더라도 다른 노드가 재연산 작업하도록 스케줄링 처리 (RDD Lineage, Automatically rebuilt on failure)
- Distributed
    - RDD에 있는 데이터는 Partition 단위로 구분되어 클러스터에 자동 분배 및 병렬 연산 수행
        - Partition 하나 당 하나의 스레드가 할당되어 처리됨
        - Partition 개수가 많다는 것은 병렬화 수준이 높다는 것
            - 병렬화 수준은 실제 사용할 수 있는 CPU Core 수에 영향을 받으므로 Partition 수를 아무리 늘려도 사용가능한 CPU Core 수에 따라 동시에 실행할 수 있는 병렬화 수준은 제한됨
- Dataset
    - 메모리나 디스크에 분산 저장된 변경 불가능한 데이터 객체들의 모음
    

## RDD 특징

1. `Immutable` : RDD는 수정이 안됨. 변형을 통한 새로운 RDD 생성
2. `Operation APIs`
    - `Transformations` : 데이터 변형, 새로운 형태로 구성된 RDD 리턴
        - map, filter, groupBy, join
    - `Actions` : 결과 연산 리턴 및 저장, RDD 내 실제 데이터를 드라이버 프로그램으로 가져오거나 내부 저장소에 결과를 저장
        - count, collect, save
3. `Lazy Evaluation` : All Transformations 
    - Transformation API 호출을 한 후 데이터 변형이 바로 실행되지는 않음
    - Transformation 단계를 Lineage로 기록했다가 필요로 하는 Action API가 호출되면 그제서야 Transformation & Action 작업 수행
4. `Controllable Persistence` : Partition 단위로 RAM / Disk에 Cache
    - 반복 조회 및 연산에 유리
    

## RDD 생성 > RDD 변형 > RDD 연산

- 외부 대용량 데이터를 읽어 최초의 RDD 생성

`LOG RDD` : 외부 데이터를 분산, 병렬로 읽음

```python
log_rdd = sc.textFile('some_file.log')
# sc = spark context
```

- 원하는 형태의 데이터가 될 때까지 다수의 RDD 변형을 반복 수행 (Transformation)

`ERR RDD` : 기존 RDD의 내용을 바꾸는 것이 아닌, 바뀐 새로운 RDD를 생성

```python
 err_rdd = log_rdd.filter(lambda line: line.startswith("ERROR"))
```

`MSG RDD`

```python
msg_rdd = err_rdd.map(lambda err: err.split("\t")[2])
```

`MEM RDD`

```python
mem_rdd = msg_rdd.filter(lambda msg: "memory" in msg)
```

- 최종 데이터를 담고 있는 RDD에 연산을 수행 (Action)

`Value`

```python
mem_rdd.count()
# 74

mem_rdd.collect()
# [mem1, mem2, mem3, ...]
# Out of Memory (OOM) 예외 발생 가능

mem_rdd.saveAsTextFile(path) 
# 외부 저장소에 저장
# RDD 내 파티션 단위로 병렬 처리되어 개별 저장됨
```
