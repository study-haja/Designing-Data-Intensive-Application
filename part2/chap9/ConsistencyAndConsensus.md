Transaction Isolation Level과 distributed consistency model은 유사점이 있지만 관심사가 서로 다르다. Transaction Isolation Level은 동시에 실행되는 트랜젝션들의 race condition을 피하는것이 관심사이지만, distributed consistency model은 딜레이와 fault를 직면하고도 리플리카들의 상태를 조정할수 있는것이 관심사이다.



## Linearizability

기반 생각은 실제로 디비가 여러 리플리카로 구성되어 있다고 하더라도, 실제로는 하나의 리플리카가 있는것처럼 동작하도록 하는 보장이다. 다른 말로 `atomic consitency` , `strong consistency`, `immediate consistency`, `external consistency`라고도 한다. linearizability는 `recency guarantee`이다. 즉, 데이터를 읽을때 항상 최신의 값을 읽을수 있어야한다.



## Linearizability VS Serializability

Serializability는 여러 트랜젝션이 동시에 여러 오브젝트들을 읽거나 쓸때, 마치 트랜젝션이 순서대로 실행된것처럼 결과를 보장해주는것을 말한다.

반면 Linearizability는 단일 오브젝트를 읽거나 쓸때, recency guarantee를 보장해주는 것이다. 여러 작업들을 트랜젝션으로 그룹화하지 않으며, 따라서 트랜젝션 챕터에 있었던 materializing conflict(lock을 미리 생성해두는 방식)방식을 제외하고 write skew문제를 해결하지 못한다.

Linearizability와 Serializability 모두 보장하는 경우를 `strict serializability` 또는 `strong one-copy serializability` 라고 한다.

그러나 serializable snapshot isolation의 경우에는 항상 일정한 snapshot을 읽기 때문에, 가장 최신 데이터를 읽는다는 보장을 할수 없다. **따라서 linearizable 하지 않다.**



## Relying on Linearizability

- Locking and leader election
  - Apache ZooKeeper, etcd와 같은 시스템은 distributed lock과 leader election을 구현하는데 사용된다.
- Constraints and uniqueness guarantees
  - 대표적으로 디비에서 uniqueness를 보장하기 위해서 사용됨.
- Cross-channel timing dependencies



## Implementing Linearizable Systems

- Single-leader replication(potentially linearizable)
  - leader or synchronously updated follower로 부터 읽으면 linearizable
- Consensus algorithm(linearizable)
- Multi-leader replication(not linearizable)
- Leaderless replication(probably not linearizable)
  - synchronously read repair를 하면 linearizable할수도 있지만, 대부분 performance 문제로 안함.



## Cost of Linearizability

Single leader replication 디비가 여러 데이터 센터에 세팅된경우, 만약 follower를 통해서 leader 데이터 센터로만 연결되는 경우 follower와 leader 데이터 센터 사이에 연결이 끊어지면 linearizable하지 않을 수 있다.

이때 linearizable을 보장하지 않으면 availability를 증가시킬수 있는데 이러한 insight를 바탕으로 CAP 이론이 탄생했다. CAP이론은 마치 network partitioning이 마치 선택적으로 일어나는것처럼 간주하는데, 실제로는 network fault는 불시에 발생할수 있다. 따라서 Network partition 되었을때 consistent와 availability중 어느것을 보장할것인가의 문제이다. 또한 CAP에서 availability의 의미는 일반적인 HA의 의미와는 차이가 있다고 한다. 또한 CAP이론은 consistency 모델중에서도 linearizability를 다루고 있고 network partition이라는 단 하나의 fault만 고려하기 때문에 실용적인 가치는 거의 없다.

실제 멀티코어 CPU에서도 linearizability를 보장하지 않는다. CPU 코어마다 캐시메모리가 할당되어 있고 메인 메모리에 있는 데이터를 각자의 캐시메모리에 복사한후 사용이 되는데, 이때 메인 메모리에 있는 데이터가 비동기적으로 캐시 메모리에 복제가 되면서 linearizability를 보장해주지 못한다. 대신 performance가 향상된다.



## Sequence Number Ordering

single leader replication의 경우, operation마다 sequence number를 증가시키며 할당하는 방식으로 인과성을 보장할수 있다. 하지만 multi leader나 leaderless replication에서는 이 방법을 적용할수가 없다. 이 경우, 노드마다 홀수나 짝수번째 수를 사용하여 sequence number를 생성하는 방식도 있고, physical clock timestamp를 사용하는 방법도 있고, 노드마다 sequence number범위를 미리 할당하는 방법도 있다. 하지만 이 모든방법이 인과성을 보장해주지는 못한다. 여기서 Google Spanner와 같은 특수한 clock의 경우는 timestamp값으로 인과성을 보장해줄수 있지만, 대부분의 clock은 이를 보장해주지 못한다.



### Lamport timestamp

operation에 (counter, node ID)값을 할당하는 방법으로, 인과성을 보장해준다.

version vector와의 차이점은, version vector의 경우는 두 작업이 동시에 일어나는것인지, 서로 인과성이 있는것인지 판단하는것이지만 lamport timestamp의 경우는 작업들간의 순서만 판단할수 있을 뿐이다. lamport timestamp는 total ordering을 정의해주지만, 해결하지 못하는 여러 문제들이 있다. 대표적으로 시스템이 user account의 유저네임을 unique하도록 보장하는 것이다. 하나의 노드에서 유저네임 설정 작업요청이 들어왔을때, 다른 노드에서 해당 계정에 대해서 유저네임을 설정하는 도중이라는 정보를 알려면 다른 노드에 요청을 보내는 수밖에 없다. 이때 네트워크 문제가 발생하면 시스템은 멈추게 되고 fault tolerant하지 않게 된다. 따라서 순서만 판단하는 것이 중요한게 아니라, 해당 순서가 언제 확정되는지도 알아야 한다. 이러한 일을 하기 위해서 total order broadcast가 필요하다.





## Total Order Broadcast

single leader replication에서 단일 CPU코어에서 작업을 처리함으로써 순서를 보장해줄수 있는데, 단일 노드에서 처리할수 있는 한계를 넘어서면 어떻게 할까? 이러한 문제를 `total order broadcast(atomic broadcast)`라고 한다. 다음 두가지 safe properties를 보장해야한다.

- Reliable delievery
  - 메시지가 유실이 되면안되고, 하나의 노드에게 메시지가 전달되면 다른 노드에게도 모두 전달되어야 한다.
- Totally ordered delievery
  - 메시지는 모든 노드에 같은 순서로 전달되어야 한다.

total order broadcast는 메시지가 전달되는 시점에 이미 순서가 결정되어있다는 점이다. 즉, 노드는 이미 도착한 메시지의 순서를 바꿀수 없다.



### total order broadcast를 사용해서 linearizable storage 구현하기

linearizable write는 보장하지만, linearizable read를 보장하려면 다음의 몇가지 옵션이 존재한다.

- 아무 메시지나 노드에게 읽기 요청을 보내고, 해당 노드가 해당 메시지를 리턴하는 시점(읽기 요청시점앞에서 다른 노드에 발생한 데이터가 모두 동기화 된 시점)에 읽기작업을 수행
- 가장 최근 메시지 위치를 받아올수 있는 경우, 해당 위치까지의 메시지가 모두 도착할때까지 기다린후 읽기작업을 수행
- 쓰기작업이 모두 동기화된 리플리카로부터 읽기작업 수행



### linearizable storage를 사용해서 total order broadcast 구현하기

알고리즘은 lamport timestamp방식과 유사하지만, sequence number에 gap을 허용하지 않는것이 차이점이다.