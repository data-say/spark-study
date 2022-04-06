## Spark Language Support

- Standalone Program
    - Scala / Java / Python / R 언어에 대한 API 제공
- Interactive Shells
    - Scala / Python / R 에 대해 interactive shell 제공
        - Repl 방식 (Read, Evaluate, Print, Loop 방식)
        - 즉각적인 분석 및 처리 작업이 필요한 **Ad hoc** 분석 작업에 용이함
- Performance
    - 일반적으로 Scala, Java 로 구현했을 때 성능이 더 좋음

## Interactive Shell

- Spark의 Interactve Shell은 로컬 실행 뿐만 아니라 Spark이 지원하는 다양한 클러스터 매니저에 연결하여 클러스터 내 분산 수행 가능

### bin/spark-shell

- Scala
    - `bin/spark-shell --master spark://spark-master-01:7177`

```scala
val df = spark.read.option("header", true).csv("hdfs://spark-master-01:9000/user/data/airline_on_time/1997.csv")
df.printSchema()
df.show()

val df2 = df.groupBy("Year", "Month").agg(count("*").as("count")).orderBy($"count".desc)
df2.printSchema()
df2.show()

df2.write.mode("overwrite").json("hdfs://spark-master-01:9000/user/result/airline_on_time/1997")

val df3 = spark.read.json("hdfs://spark-master-01:9000/user/result/airline_on_time/1997")
df3.printSchema()
df3.show()
```

### bin/pyspark

- Python
    - `YARN_CONF_DIR=/user/spark3/conf2 bin/pyspark --master yarn`

```python
df = spark.read.csv("hdfs://spark-master-01:9000/user/data/airline_on_time/1998.csv", header="true")
df.printSchema()
df.show()

import pyspark.sql.functions as func
df2 = df.groupBy("Year", "Month").agg(func.count("*").alias("count")).orderBy(func.col("count").desc())
df2.show()
```

### bin/sparkR

- R
    - `HADOOP_CONF_DIR=/user/spark3/conf2 bin/sparkR --master yarn`

```r
df <- read.df("hdfs://spark-master-01:9000/user/data/airline_on_time/1998.csv", "csv", header="true")
printSchema(df)
showDF(df)

showDF(orderBy(df2 <- summarize(groupBy(df, df$Year, df$Month), count=n(df$"*")), desc(df2$count)))
```

### bin/spark-sql

- sql
    - `bin/spark-sql --master spark://spark-master-01:7177`

```sql
select count(*) from csv.'hdfs://spark-master-01:9000/user/data/airline_on_time/*.csv';
select * from csv.'hdfs://spark-master-01:9000/user/data/airline_on_time/*.csv' limit 2;

describe select * from csv.'hdfs://spark-master-01:9000/user/data/airline_on_time/*.csv' limit 2;

create temporary view temp_view_airline using csv options (path="hdfs://spark-master-01:9000/user/data/airline_on_time/*.csv", header="true");
show views;
describe temp_view_airline;

select Year, Month, count(*) as count from temp_view_airline group by Year, Month order by count desc;
```