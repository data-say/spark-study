## How Spark works

- **DAG**: RDD에 대한 operator 들의 flow로 구성된 그래프
    - 실행을 위해 단계별 stage로 분할됨
    - 각 stage는 task들로 구성됨
        - 분산 병렬 실행을 위해 cluster에 제출됨
    - RDD 하나의 partition에 하나의 task가 할당됨
    - worker의 executor process는 전달받은 task들을 실행
    - stage를 구성하는 모든 task들의 실행이 끝나면, 다음 stage의 task들이 실행을 위해 cluster에 제출됨

## DAG Scheduler

- Apache Spark은 task 수행을 위한 workflow 같은 그래프 생성
- RDD에 수행하는 여러 function에 대해 자동으로 pipeline으로 작업 처리
- 성능 향상을 위해 되도록 가까운 곳에 있는 데이터 partition에 대해 task를 수행하도록 할당 조정
- 성능 상 많은 비용을 요구하는 `shuffle` 을 피하기 위해 partition 인식
    - partition이 이미 그룹핑 되어 있다면, 불필요한 shuffling 작업 수행 X

## DAG

```scala
sc.textFile("/wiki/pages-articles") // RDD[String] 생성
// 파일 크기(파일 block 개수)에 따라 partition 개수가 정해짐
  .map(line => line.split("\t")) // RDD[Array[String]]
  // map: 할당된 partition에 대해서만 처리하는 api이므로 partition 수 변화나 이동 없음
  .map(arr => (arr(0), arr(1).toInt)) // RDD[(String, Int)]
  // 마찬가지로 partition 내 레코드에 대해서만 변환 작업 수행
  .reduceByKey(_+_.3) // RDD[(String, Int)]
  // tuple의 key별로 데이터를 그룹핑해서 같은 key내 value에 대해 reduce 연산 수행
  // partition 간 레코드 이동이 필요한 shuffle 연산 발생
  // RDD partition 개수 변함 (3개로 명시)
  .collect() // Array[(String, Int)]
  // collect: RDD의 모든 레코드들을 executor에서 driver로 array형태로 묶어 가져옴
```

## Execution Plan

- DAG는 shuffle을 기준으로 분할됨
    - stage 내에서는 partition 변화 없으며 RDD는 같은 개수의 partition 을 가짐
    - stage 내 여러 operator들은 하나의 pipeline을 구성하여 순차적으로 실행됨
- Example
    - Stage 1
        1. Read HDFS split
        2. Apply both the maps
        3. Start Partial reduce
        4. Write shuffle data
    - Stage 2
        1. Read shuffle data
        2. Final reduce
        3. Send result to driver program

## Stage Execution

- 각 Task들은 executor로 전달되기 위해 직렬화됨
    - ex) `Read HDFS split > Maps > Partial Reduce > Write Shuffle Data`
- Spark은 task들의 실행을 스케줄링, 실제 실행을 위해 task를 executor로 전달
    - Spark engine 내부에서 자동적으로 실행됨

## Executor, Task

- Executor 는 전달받은 Task들을 실행함
    - 하나의 executor에 core가 3개 할당되어 있다고 가정했을 때, 해당 executor는 동시에 3개의 task까지만 병렬로 실행 가능
    - 3개씩 실행 후 task 실행이 끝난 core는 바로 다음 task 순차적으로 실행
    

## Summary of Components

1. RDD
    - partition 단위로 데이터 나뉘어짐
    - parition 별로 task가 하나씩 할당되어 실행시 여러 task가 동시에 병렬로 실행됨
    - `map` , `filter` 와 같은 operatior 제공
2. DAG
    - RDD operation들에 대한 논리적인 실행 그래프 생성
    - 실제 수행을 위해 stage 단위로 분할됨
3. Stage
    - Stage를 구성하는 task들은 병렬로 실행됨
4.  Task
    - Executor에 의해 병렬로 실행됨
    - Executor가 할당받은 core 개수에 따라 동시에 병렬로 수행될 수 있는 task 수 결정됨