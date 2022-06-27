## RDD - Fault Tolerance

- **Lineage**
    - RDD는 일련의 Transformation 작업들을 추적함

## RDD - Fault Recovery Test

Example

- RDD 결함 복구 기능
    - Cache > Iterate Actions > Crash(Fault) > Recompute
    - Lineage를 통해 손실된 cache 데이터를 복구: 손실된 일부 데이터를 disk에서 다시 읽어오기 때문에 시간 다소 소요됨

```scala
val allRDD = sc.textFile("hdfs://...")
val targetYearRDD = allRDD.filter(_.startswith("1997"))
val carrierRdd = targetYearRDD.map(_.split('.')(8))

// lineage 정보 확인
carrierRDD.toDebugString
carrierRDD.cache()

carrierRDD.filter(_.contains("WN")).count() // 실제 cache 이루어짐
carrierRDD.filter(_.contains("AS")).count() // 빠름

// kill -9 1723 -- executor 의도적으로 kill
carrierRDD.filter(_.contains("AS")).count() // 다시 읽어와 연산하므로 속도 증가, Input 사이즈도 증가
/* 
locality level
PROCESS_LOCAL: task가 돌고 있는 같은 executor process 내 메모리 cache 읽음
NODE_LOCAL: 같은 노드의 disk에서 데이터를 읽음
RACK_LOCAL: 같은 rack 장비의 다른 노드의 disk에서 네트워크를 통해 데이터를 읽음
*/
// 재연산된 데이터는 메모리에 다시 cache 되어 복구됨
```