## Administrative Web UIs

1. Driver (Spark Application 자체 Web UI)
    - Driver가 기동중일때만 볼 수 있음
2. History Server
    - Driver 종료 후에도 관련 UI를 볼 수 있도록 제공하는 UI
3. Cluster Manager (Cluster Resource Manager)
    - Spark Standalone, Hadoop YARN 에서 제공
    

## Driver (Spark Application) Web UI

- Spark Context가 제공하는 UI
- Spark Applicaton이 실행되는 동안의 다양한 정보를 보여주는 대시보드 역할을 함
- 확인할 수 있는 정보(메뉴)들
    - Spark이 사용하는 각 작업의 scheduling 정보
    - Memory와 같은 resource 사용량
    - 각종 Environmental information
    - 분산환경에서 기동중인 executor 정보
- Default port: `http://<Driver Node>:4040`

```scala
sc.uiWebUrl

val rdd = sc.textFile("hdfs:spark-master-01:9000/user/data/airline_on_time")
rdd.take(2)

val rdd2 = rdd.filter(line => line.split(",")(3).equals("1"))
rdd.count() - rdd2.count()

rdd2.cache() # 캐시 저장하면 훨씬 빨라짐
rdd2.count()
rdd2.count()

rdd2.filter(line => line.split(",")(8).equals("WN")).count()
rdd2.filter(line => line.split(",")(8).equals("DL")).count()
rdd2.map(line => line.split(",")(8)).countByValue()

rdd2.unpersist() # 캐시 해제

sc.stop() # Spark Context 종료
sc.isStopped

val sc = new org.apahe.spark.SparkContext("yarn", "SparkShell_YARN_NEW")
sc.uiWebUrl
```

## History Server Web UI

- 이미 실행이 끝난 Spark Application의 실행내역을 보기 위함
- `.inprogres`로 끝나는 이벤트 로그: 아직 실행중이거나 비정상 종료된 spark application의 이벤트 로그
- Default port: `http://<History Server>:18080`

## Spark Standalone Web UI

- 빠르고 가벼운 Spark Application 전용 Cluster Manager가 필요할 때 사용
- Default port: `http://<Standalone Master>:8080`

## Hadoop YARN Web UI

- MapReduce 뿐만 아니라 Cluster 환경에서 구동 가능한 다양한 Application 플랫폼 지원
- Spark, Hive, HBase, Presto, Flink 등의 Application 실행 가능
- Default port: `http://<ResourceManager>:8088`
 