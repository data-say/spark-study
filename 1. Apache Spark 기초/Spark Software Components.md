## Spark Software Components

- **Spark Driver**
    - main program 자체는 Driver process로 실행됨
    - Client side application으로서 프로그램 시작점인 Spark Context를 내부에 생성하고 유지함
- **Spark Context**
    - 주요 데이터 처리 코드는 Spark Executor process 내에서 task 형태로 실행됨
    - Driver는 Spark Context를 통해 Spark standalone, Cluster Manager(ex. Hadoop YARN)와 통신하여 실제 분산 병렬 연산을 수행하는 Executor 생성을 요청
- Task (데이터 처리 연산)는 local 환경, cluster 환경에서 thread 단위로 동시에 병렬로 수행됨
    - Local 환경: 단일 machine에서 local cpu core를 통해 병렬 thread로 여러 task 수행
    - Cluster 환경: 복수개의 machine에서 각 서버의 cpu core를 통해 다수개의 task들을 분산&병렬로 수행
- Spark Application은 하나의 driver program과 복수개의 executor program으로 구성되어 실행됨
    - 각 executor들은 driver가 할당한 task들을 병렬로 실행하고 그 결과를 driver에게 전달
    - Spark Application을 local 환경에서 실행하면 executor program은 별도로 실행되지 않고 driver program의 process 내에서 local thread로 실행됨
    - Spark Application을 분산(cluster) 환경에서 실행하면 executor program은 cluster manager가 할당해준 서버들 위에서 각각 별도의 process로 실행됨
    - Driver process와 Executor process은 JVM(자바 가상 머신) process 형태로 실행됨

## Submitting Applications on Cluster

1. Program을 Cluster 환경으로 submit 하면, Driver Program 내의 Spark Context가 지정한 Cluster manager에 연결되어 Executor 생성을 요청함
2. Cluster manager는 지정한 만큼의 cpu core와 memory 크기를 할당하여 Cluster 내 적절한 노드 위에서 요청한 개수만큼의 Executor process를 생성함
3. 생성된 Executor들은 Driver Program에게 접근하여 통신을 가능하게 함. 이후 Driver Program은 할당받은 Executor들과 통신하며 각 Executor가 수행할 task들을 scheduling하고, 각 Executor에게 적절한 task들을 전달하고, 그 실행 결과를 전달받음. 이때 task 수행에 필요한 application code들은 JAR, Python, R file 형태로 executor로 전달됨.

## Example

```bash
## Spark Master / Worker-01 / Worker-02 / Worker-03
jps # java process 확인

## Spark Master
# ResourceManger
# NameNode
# Jps
# SecondaryNameNode

## Worker-01 / Worker-02 / Worker-03
# NodeManager
# DataNode
# Jps
```

```python
## vi /user/spark3/word_count.py

from pyspark.context import SparkContext

sc = SparkContext(appName = "word_count")
text_file = sc.textFile("hdfs://spark-master-01:9000/user/data/airline_on_time/*.csv")

counts = text_file.flatMap(lambda line: line.split(",") \
				  .map(lambda word: (word, 1) \
					.reduceByKey(lambda a, b: a * b)
counts.saveAsTextFile("hdfs://spark-master-01:9000/user/result/airline_on_time_word_count")
```

```bash
## Spark Submit을 통한 배포
# Executor 개수, Executor 당 Core 수, Executor 당 Memory 크기 설정

YARN_CONF_DIR=/user/spark3/conf2 ./bin/spark-submit --master yarn --num-executors 5 --executor-cores 2 --executor-memory 2G /user/spark3/word_count.py
```

```bash
## Spark Master / Worker-01 / Worker-02 / Worker-03
jps # java process 확인

## Spark Master
# ResourceManger
# NameNode
# SparkSubmit
# Jps
# SecondaryNameNode

## Worker-01 / Worker-02 / Worker-03
# YarnCoarseGraineExecutorBackend ## spark executor process
# NodeManager
# DataNode
# Jps
```

```bash
## Spark Application 관련 파일 확인 (application id)
sudo find /tmp -name [APPLICATION_ID]
# /tmp/hadoop-spark/nm-local-dir/usercache/spark/appcache/APPLICATION_ID
# /tmp/hadoop-spark/nm-local-dir/nmPrivate/APPLICATION_ID
```
