## Apache Spark 개요

`Apache Spark`: 빅데이터에 대한 컴퓨팅 연산을 다수의 서버로 구성된 클러스터에서 분산, 병렬 처리하기 위한 오픈소스 엔진

- 싱글노드에서 분산이 아닌 병렬 처리 엔진으로도 사용 가능

1. `Unified Engine` : 하기 작업들을 개별 엔진이 아닌 하나의 엔진에서 처리할 수 있는 통합 데이터 처리 엔진
    - Batch
    - SQL
    - Streaming
    - ML
    - Graph
2. `High-level APIs` : 쉽게 제작하기 위한 상위 레벨의 API 제공하며 지속적으로 최적화됨
3. `Integrate Broadly` : 저장소가 아닌, 외부의 데이터를 읽어 분산 환경에서 원하는 형태로 데이터를 처리한 후, 그 결과를 다시 외부에 저장하는 컴퓨팅 엔진
    - Cloud Storage (hadoop의 HDFS, AWS S3 등), NoSQL (Cassandra 등), RDBMS (MySQL 등) 의 다양한 형태의 외부 저장소와 연동을 지원
    

## Apache Spark 설명

- 대용랑 데이터 프로세싱을 위한 빠르고 범용적인 인메모리 기반 클러스터 컴퓨팅 엔진
    - 보다 빠른 연산을 위해 클러스터 내 여러 노드의 메모리에 반복 조회가 필요한 데이터를 캐시하는 기능 제공
- 분산 메모리 기반의 빠른 분산 병렬 처리
- 배치, 대화형 쿼리, 스트리밍, 머신러닝과 같은 다양한 작업 타입을 지원하는 범용 엔진으로 Apache Hadoop과 호환
- Scala, Java, Python, R, SQL 등의 High-level API 제공
    - 데이터 처리하기 위한 어플리케이션 소스코드 작성 시 분산, 병렬 처리를 직접 고민하지 않아도 됨
- `클러스터 매니저`
    - 클러스터 환경에서 분산, 병렬 연산이 가능하도록 설계되어 있으나 클러스터 내 서버들의 CPU 메모리의 할당, 회수와 같은 리소스 관리를 직접 하는 것은 아님
    - 다수의 노드로 구성된 클러스터의 리소스를 관리하는 클러스터 매니저 존재
    - CPU Core 수, 메모리 크기, Execute 개수 (분산 환경에서 병렬로 연산을 수행하는 데 쓰임) 등을 클러스터 매니저에게 요청하면 클러스터 내 관리하고 있는 사용 가능한 노드의 자원을 적절히 할당
    - ex) Hadoop YARN, Apache Mesos, Kubernetes
    - 외부 클러스터 매니저가 없어도 별도의 Spark 어플리케이션 전용 클러스터 매니저인 Spark Standalone 제공
- `Spark Core`
    - 분산, 병렬 연산을 위한 작업 스케줄링 및 Fault tolerance (오류 발생 시에도 문제 없이 연산을 지속할 수 있음) 등의 핵심 기능 제공
    - 기본 데이터 모델인 RDD를 기반으로 데이터 연산을 분산, 병렬로 처리
    - 다양한 확장 라이브러리 동작
        - ex) Spark SQL, Spark Streaming, MLlib (Machine Learning), GraphX (Graph processing), SparkR (R on Spark), PySpark (Python on Spark)

## Apache Spark 특징

- In-Memory 컴퓨팅
    - Disk 기반도 가능
    - 빠른 데이터 Processing
        - In-Memory Cached RDD는 최대 100배 빠름
- RDD (Resilient Distributed Dataset) 데이터 모델
- 다양한 개발 언어 지원
    - Scala, Java, Python, R, SQL
- 대화형 질의를 위한 Interactive Shell
    - Scala, Python R Interpreter
- Rich APIs 제공
    - 80여개 이상의 다양한 API 제공
- General execution graph ⇒ DAG (Directed Acyclic Graph) ⇒ Multiple stages of map & reduce
- Hadoop과 유연한 관계 (HDFS, HBase, YARN 등)
- Real-time Stream Processing
- 하나의 어플리케이션에서 배치, SQL 쿼리, 스트리밍, 머신러닝과 같은 다양한 작업을 하나의 워크플로우로 결합 가능