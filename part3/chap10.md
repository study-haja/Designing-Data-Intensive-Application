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