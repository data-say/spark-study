## RDD - Lazy Evaluation + No Cache

Log Mining Example

- 로그 파일로부터 에러 메시지만 불러와 확인하기

```scala
lines = sc.textFile("hdfs://...") // base RDD
errors = lines.filter(_.startsWith("ERROR")) // transformed RDD
messages = errors.map(_.split('\t')(2))

// action #1
messages.filter(_.contains("foo")).count 
/*
action api 호출 되어 transformation api 순차 실행됨
driver가 대상 데이터의 위치와 크기 확인
block 하나 당 하나의 task를 생성하여 executor로 보냄
task: read > filter > map > filter > count
각 executor는 task 실행 결과를 driver로 보냄
*/

// action #2
messages.filter(_.contains("bar")).count
/*
driver가 대상 데이터의 위치와 크기 확인
block 하나 당 하나의 task를 생성하여 executor로 보냄
executor들은 각기 전달받은 task 실행
각 executor는 task 실행 결과를 driver로 보냄
*/

// 동일한 action 수행을 하더라도 처음부터 disk에서 읽어와 반복 진행
```

## RDD - Lazy Evaluation + Cache

- 성능향상을 위해 메모리에 cache

```scala
lines = sc.textFile("hdfs://...") // base RDD
errors = lines.filter(_.startsWith("ERROR")) // transformed RDD
messages = errors.map(_.split('\t')(2))
messages.cache() // cached RDD

// action #1
messages.filter(_.contains("foo")).count 
/*
action api 호출 되어 transformation api 순차 실행됨
driver가 대상 데이터 block의 위치와 크기 확인
block 하나 당 하나의 task를 생성하여 executor로 보냄
task: read > filter > map > cache > filter > count 
partiton 단위로 메모리에 cache
각 executor는 task 실행 결과를 driver로 보냄
*/

// action #2
messages.filter(_.contains("bar")).count
/*
driver는 task를 executor로 보냄
cache 된 데이터로부터 연산 시작
task: filter > count 
각 executor는 task 실행 결과를 driver로 보냄
*/

// 동일한 action 수행을 하더라도 처음부터 disk에서 읽어와 반복 진행
```

## RDD - Behavior with Less RAM

- Executor의 메모리가 부족하여 모든 데이터가 다 cache 되지 않을 경우
    - cache는 RDD의 partition 단위로 처리됨
    - 하나의 partition을 모두 cache하기에 메모리가 부족하다면 partition의 일부만 cache X (하나의 parition `all cache` or `not cache`)
    - disk로부터 다시 읽어와야 함