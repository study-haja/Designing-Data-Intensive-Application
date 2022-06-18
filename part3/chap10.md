# Batch Processing

일단 요청과 응답관점으로 시스템의 유형을 분류해보자.

- Services(online systems)
  - 응답시간이 서비스의 성능을 측정하는데 있어서 가장 중요한 measure가 됨.
  - availability가 매우 중요함.
- Batch processing system(offline systems)
  - 보통 대용량 데이터를 입력으로 받아서 처리하고 결과를 도출하는 시스템
  - 실행시간이 오래걸리므로 해당 결과를 기다리는 유저는 없음
  - 대신에 batch job들이 주기적으로 실행되고 throughput이 성능을 측정하기 위한 measure
- Stream processing system(near-real-time-systems)
  - 위의 두 시스템의 중간과 같은 형태라고 볼수 있는데, batch 시스템과 비슷하지만 더 낮은 latency를 제공



## MapReduce and Distributed FileSystems

Unix tool들이 stdin, stdout을 입력과 출력으로 사용하는 반면에, MapReduce는 HDFS라는 분산 파일 시스템에 파일들을 기록한다.

HDFS는 NAS나 SAN 아키텍쳐와는 달리 shared-nothing principal에 기반하고 있고, special hardware가 필요없다. HDFS에서 각각의 노드는 데몬 프로세스를 실행하고 있으며 다른 노드가 접근할수 있도록 네트워크 서비스를 실행하고 있다. Namenode는 어느 머신에 파일블락들이 위치해있는지 트래킹하고 있다.

HDFS는 또한 스케일 아웃이 잘되며 commodity hardware를 사용할수 있어서 비용도 저렴한 편이다.

#### MapReduce Job Execution

MapReduce job을 생성하기 위해서, 두가지 콜백을 구현해야 한다.

- Mapper
  - mapper는 각각의 input record에 대해서 한번 호출되고, input record에서 key,value를 뽑아내는 작업을 수행한다.
- Reducer
  - Mapper가 생산한 데이터에서 key가 같은 데이터들을 대상으로 reducer로직을 수행한다.



#### Distributed execution of MapReduce

- 프레임워크단에서 머신들사이에 데이터를 옮기는(셔플과정)의 복잡도를 다룬다.
- 맵리듀스에서 배치잡들의 아웃풋은 잡이 성공을 했을때만 valid하기 때문에, 다양한 workflow 스케줄러(Oozie, Azkaban, Airflow, ...)들을 사용해서 이러한 잡 디펜던시들을 관리할수 있다.
- 보통 MapReduce 잡에서 데이터를 처리하면서 조인작업을 처리하기 위해서 디비에 쿼리하는 작업은 네트워크 round trip time때문에 비효율적일뿐만 아니라 디비성능에도 좋지 않으므로, 디비에 있는 데이터를 HDFS로 복사하는 작업을 보통 수행 한다.



#### Reducer-Side Joins

- input의 key를 기준으로 mapper output을 생성하고, key별로 소팅된 output들이 reducer로 전달되고 reducer는 merge를 수행 -> Soft-merge join 알고리즘이라고도 알려져있음.
- 특정 key에 해당하는 데이터가 너무 많을경우, 특정 reducer에만 작업이 skew될수 있는경우, 해당 키에 해당하는 데이터는 여러 reducer에게 분산하는 알고리즘이 사용된다. 이 경우, 모든 reducer에게 join에 사용되는 데이터를 복제해야 한다는 단점이 있다.



#### Map-Side Joins

- reducer-side join의 장점은 인풋 데이터에 대한 특별한 가정없이도 잘 동작한다는 것이다. 하지만 데이터를 소팅,복사,머지 과정이 비싸다는 단점이 있다.
- 만약 인풋 데이터에 특정한 가정을 할수 있다면, map-side join을 사용해서 성능을 향상시킬수 있다. 
- Broadcast hash joins
  - large dataset과 small dataset을 서로 조인할때 사용할수 있는 방법이다.
  - small dataset을 읽어서 메모리에 해시테이블을 로드하고, large dataset partition에 있는 데이터와 조인하는 방법이다.
  - 모든 mapper에 해시테이블을 복제해야한다는 단점이 존재
  - 해시테이블을 사용하는 대신에 local disk에 small join input에 대한 read-only index를 구축하는 방법또 가능한데, 이 경우 os의 page cache를 이용해서 속도도 해시테이블만큼 빠를뿐 아니라, 메모리에 해시테이블을 올릴수 없는 경우까지도 커버할수 있다.
- Partitioned hash joins
  - mapper에서 자신이 조인할 두 데이터셋의 일부 파티션만 로드하는 방법
  - mapper에서 상대적으로 적은 양의 데이터만 로드할수 있다는 장점
  - 조인 인풋들이 같은 수의 파티션으로 구성되어있고, 각각의 데이터가 같은 키와 같은 해시함수를 기반으로 파티션에 맵핑되어 있는 경우에만 사용가능
  - Hive에서 bucketed map joins라고 알려져있음.
- Map-side merge joins
  - 인풋 데이터가 Partitioned hash joins 방식으로 파티션 되어있을뿐 아니라, key별로 정렬까지 되어있는 경우 가능한 join
  - 인풋 파일들을 순차적으로 읽어서 join이 가능
  - map-side merge joins이 가능하려면, 인풋 데이터가 미리 key별로 정렬되어있어야 하므로 보통 이전의 job의 reducer에서 처리될수 있었을것지만, 보통 map-side merge join후에 reducer에서 다른 목적으로 한번더 로직을 처리할수 있기때문에 map-side merge join이 더 나을수 있다.
- MapReduce에서 join의 성능을 높이려면 데이터셋의 physical layout을 잘 이해해야 한다. 파티션의 수뿐만 아니라 데이터가 파티션되고 정렬되는 기준이 되는 키들을 알아야 한다. Hadoop에서 이러한 종류의 메타데이터는 HCatalog와 Hive metastore에서 유지된다.



## The Output of Batch Workflows

- index를 생성하는 경우
  - 전체 document들을 대상으로 생성하는 방법과 incremental 하게 생성하는 방법(Lucene처럼 추가 segment file을 기록하고 비동기적으로 머지하는 방식)
- key-value store
  - join 관점에서 레코드 하나당 결과를 디비에 쓸때 네트워크 요청이 발생하므로 오버헤드가 크다. bulk로 업데이트한다고 해도 성능이 좋지 않다.
  - mapper나 reducer에서 병렬로 디비로 요청을 보내면 디비의 성능이 영향을 받는다.
  - 더 좋은 방법은 결과파일을 분산 파일 시스템에 기록하고, key value store 서버에 로드하는 것이다.  로드가 완료되면 서버는 새로운 파일을 사용하도록 변경한다.



## Comparing Hadoop to Distributed Databases

앞에서 다루었던 MapReduce의 기능들은 사실 `massively parallel processing`(MPP) 데이터베이스에서 이미 구현이 다 되었다. MPP와 MapReduce의 차이점은 MPP는 클러스터에 분석 SQL 쿼리들을 병렬로 실행하는데 초점을 맞추고 있고, MapReduce는 임의의 프로그램을 클러스터에 실행하는 시스템이라는 점이다.

Hadoop은 데이터를 HDFS에 차별없이 적재하고 나중에 어떻게 처리할지를 고민하는 디자인이라면 MPP의 경우는 데이터를 import하기 전에 데이터와 쿼리 패턴에 대한 모델링을 주의깊게 해야하는 디자인이다.

### Processing models

processing중에는 SQL로 표현할수 없는 것도 있다. MapReduce는 SQL로 표현할수 있는 처리들을 지원하지만 이 둘로도 표현할수 없는 처리들이 있다. Hadoop 에코시스템들은 HBase같은 random-access OLTP 디비도 포함하고, MPP 스타일의 analytic 디비인 Impala를 포함한다.

### Designing for frequent faults

배치 프로세스들은 보통 유저들에게 즉각적인 영향을 주지 않는다. 보통 MapReduce에서 하나의 테스크가 fail이 나면 모든 잡을 다시 실행하는것은 낭비기 때문에 중간 결과들을 디스크로 기록하는 과정이 존재한다. 하지만 이 과정이 job의 속도를 느리게 만든다. 원래 MapReduce가 디자인 될때, 같은 머신에서 여러 배치잡들이 실행될수 있고 우선순위에 따라서 하나의 잡이 다른잡에 의해서 선점될수 있음을 염두해두었기 때문에 디스크로 기록하는 과정이 존재한것이라고 한다. 현재 쿠버네티스, 메소스, YARN에서는 general priority preemption을 지원하지 않는다.



## Beyond MapReduce

MapReduce 잡을 작성하는 것은 어렵고 피곤한 일이기 때문에 higher-level programming models(Pig, Model, Cascading, Crunch)들이 MapReduce 상위 레벨 추상화 레이어로서 생성되었다.



### Materialization of Intermediate State

대부분의 경우 job의 결과는 intermediate state를 나타내는 경우가 많다. 이러한 intermediate state를 파일로 기록하는 과정을 `materialization` 이라고 한다. 이 과정은 아래의 단점이 있다.

- MapReduce job은 오직 이전이 job들이 모두 실행이 끝난후에야 실행될수 있다.
- mapper의 코드는 사실 이전의 reducer에서 기록한 파일을 읽는 부분을 포함하기 때문에 이전의 reducer로직에서 처리될수 있는 부분이다.
- 분산 파일 시스템에 intermediate state를 기록하는 하면 다른 노드에도 이를 복제해야 하기 때문에 오버헤드가 발생한다.

이런 문제를 해결하기 위해 나온 엔진이 Spark, Tez, Flink 등이다. 이 엔진들의 공통점은 전체 workflow를 하나의 job으로 처리하는 것이다. MapReduce와 비교했을때 아래의 장점이 있다.

- 꼭 필요한 경우만 sorting작업을 할수 있다. MapReduce에서는 sort작업은 강제된다.
- mapper에서 한 작업이 이전의 reduce operator에 포함될수 있다.
- workflow에 정의된 join과 데이터 의존관계들이 명시적으로 정의되어 있으므로 locality optimization이 가능하다.
- intermediate state를 HDFS에 기록할 필요없이 로컬 디스크나 메모리에 유지할수 있다.
- 오퍼레이터들은 인풋이 준비가 되면 즉시 시작할수 있다.
- task들이 JVM 프로세스를 공유할수 있다.

### Fault tolerance 

fault발생시 해당 데이터를 다시 계산하는것이 항상 좋은 해결책인것은 아니다. 만약 intermediate data가 source data에 비해서 훨씬 작거나 계산과정이 CPU-intensive라면 파일로 materialization하는것이 더 좋을수 있다.



### Graphs and Iterative Processing

graph 알고리즘들은 cross-machine 커뮤니케이션을 많이 필요하므로 네트워크 비용으로 인해서 속도가 많이 느려진다. 따라서 단일 노드에 메모리나 디스크에 그래프를 적재할수 있다면, 그렇게 하는것이 더 빠르다. 하지만 그렇게 하지 못하는 경우 Pregel을 사용해야 한다.



Batch processing systems들은 점점 high-level declarative operator들을 지원하고 MPP 디비들은 더 programmable, flexible 해짐에 따라서 두 시스템은 점점 유사해지고 있다.
