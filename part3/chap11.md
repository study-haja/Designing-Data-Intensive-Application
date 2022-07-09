# Stream Processing

메시지 브로커에는 두가지 타입이 있다.

- JMS/AMQP 
  - 컨슈머가 메시지를 처리하면 해당 메시지가 영구적으로 삭제된다.
  - 메시지가 produce된 순서대로 처리된다는 보장이 없다.
  - 각각의 메시지 처리비용이 크고, 처리 순서가 중요하지 않을때 효과적이다.
- Log based message broker
  - 단일 파티션내에서 메시지 처리순서가 보장된다.
  - 하나의 토픽을 처리할수 있는 컨슈머는 파티션수로 제한된다.
  - 각각의 메시지 처리비용이 낮고, 처리 순서가 중요할때 효과적이다.



여러 시스템들간에 데이터를 싱크하기 위한 방법으로 dual write 방식이 있다. dual write는 어플리케이션에서 여러 데이터 시스템에 쓰기작업을 수행하는 것이다. 하지만 이는 2가지 문제가 있다.

- race condition
  - 여러 어플리케이션의 쓰기작업이 동시에 일어나면서 최종상태가 inconsistent해진다.
- partial success(or failure)
  - 여러 쓰기작업도중 일부만 성공할수 있다.

이를 해결하기 위해 CDC가 사용될수 있다. CDC대신에 트리거를 사용할수도 있지만, 이는 오류가 발생하기 쉽고 상당히 디비에 오버헤드를 유발한다.



## Event Sourcing

- CDC와 유사하게 어플리케이션 상태에 대한 모든 변화를 로그에 저장하는것이고, CDC와 가장 큰 차이점은 추상화 레벨에 적용된다는 점이다.
- 로우 레벨 상태보다는 어플리케이션 레벨에서 발생하는 이벤트들을 트래킹한다.
- 원인을 알아내거나 디버깅하기 용이하게 만든다.
- CDC의 경우 같은 key를 가진 메시지들을 compaction할수 있지만, event sourcing의 경우 이벤트들의 full history가 필요하므로 compaction할수 없다.
- event sourcing에서는 command와 event를 구분하는데, command는 유저가 보낸 요청을 의미하고, event는 validation이 성공적이고 수용된 command를 의미한다.

## State,Streams, and Immutability

### Advantages of Immutable events

- 이벤트들에 대한 기록이 append-only-log에 있으므로, 문제가 발생시 원인을 파악하기 쉽고 복구하기도 쉽다.
- 현재 상태뿐 아니라 이전의 변경내역이 기록되어 있으므로, 더 많은 정보를 제공해준다.



### Limitations of immutability

- update나 delete가 자주 발생하는 워크로드의 경우, 로그가 너무 많이 쌓이기 때문에 compaction,garbage collection의 성능이 중요하다.
- 법적인 이유로 immutable 데이터를 반드시 삭제해야하는 경우, 삭제하기가 어렵다.
  - 특정 데이터를 삭제한다는 이벤트를 로그에 추가하는것으로 충분하지 않고, 로그 자체를 다시 작성해야한다.
  - 스토리지 엔진, 파일 시스템, SSD에 데이터를 쓰면 보통 백업데이터도 함께 저장되어 완전히 삭제하는것이 어렵다.



## Processing Streams

### Stream 처리의 세가지 옵션

1. DB, cache, search index등에 기록
2. 유저에게 메일을 보내거나 push알림을 보내기
3. 한개 이상의 input stream을 받아서 여러 output stream을 생성하는 방법

책에서는 3번째 옵션을 주로 다룸.



### Stream Processing의 활용

- Complex Event Processing(CEP)
  - 특정 이벤트 패턴들을 검색하는 어플리케이션들을 위한 이벤트 스트림 분석 방법
  - SQL과 같은 high-level declarative query 언어를 사용
- Stream Analytics
  - CEP와의 경계가 불분명하지만, 일반적으로 특정 이벤트 패턴을 검색하기보다는 time window동안에 여러 이벤트들을 대상으로 집계하고 통계적 metric을 뽑아내는것을 지향
- Maintaining materialized views
  - 시작부터 현재까지의 이벤트들을 유지해서 디비나 어플리케이션에 변경분을 반영할때 사용하는 방법
- Search on streams
  - full-text search 쿼리와 같은 복잡한 기준에 기반하여 개별 이벤트들을 검색할때 사용





## Reasoning About Time

> Stream 처리에서는 보통 타임윈도우를 결정하기 위해서 local시간을 사용하는데, 이는 간단하다는 장점이 있으면서도 이벤트 생성시점과 처리시점의 시간 간격이 길어지면 문제가 될수 있다.

### knowing when you're ready(window가 끝난 시점 알기)

- 특정 타임 윈도우에 해당하는 데이터가 반드시 특정 시간 안에 도착하리라는 보장이 없으므로 다음 두가지 옵션이 존재
  - 이때 뒤늦게 도착하는 이벤트들은 무시
  - 뒤늦게 도착하는 이벤트들을 가지고 이전 윈도우 아웃풋을 수정
-  타임 윈도우의 끝을 알리는 특별한 메시지를 포함할수도 있지만, 만약 여러 머신에 여러 프로듀서들이 메시지를 보내는 상황에서는 프로듀서의 추가와 삭제가 어려울수 있다.

### Whose clock are you using, anyway?

클라이언트 로컬 시간을 기준으로 타임스탬프를 정할수 있지만, 클라이언트의 시계를 신뢰할수 없다. 그렇다고 서버가 요청을 수신한 시간을 사용하면 user interaction에 대한 정보가 사라져 버린다.

이를 보완하기 위해서 log three timestamp를 사용할수 있다.



- 디바이스 시계기준으로 이벤트 발생 시각
- 디바이스 시계기준으로 이벤트 전송 시각
- 서버 시계기준으로 이벤트 수신 시각

두번째 시간과 세번째 시간의 차이를 첫번째 시간에 적용함으로써 실제 이벤트 발생 시간을 추정할수 있다.



### Types of windows

- Tumbling window : 모든 이벤트가 단 하나의 윈도우에만 속하는 고정된 크기의 윈도우
- Hopping window : 이벤트가 여러 윈도우에 속하는 고정된 크기의 윈도우
- Sliding window : 특정 간격안에 발생하는 모든 이벤트들을 포함하는 가변적 크기의 윈도우
- Session window 
  - 고정된 duration자체가 없고 같은 유저에 대한 모든 이벤트들을 그룹핑하는 윈도우
  - 일정 시간동안 유저의 활동이 없으면 윈도우 종료



## Stream Joins

- stream-stream join(Window join)
  - 두개의 stream의 윈도우 크기만큼의 데이터를 대상으로 조인하는 방법 
- stream-table join(stream enrichment)
  - 대표적으로 stream에 있는 이벤트와 연관된 데이터를 DB에 쿼리해서 조인하는 방식을 예로들수 있음.
  - DB의 local copy가 메모리에 로드가능하다면, 로드해서 DB의 부하를 줄일수 있음.
  - 다만 DB의 상태가 바뀔수 있는데 이 경우, change data capture로 해결할수 있다.
    - DB의 변경분을 실시간으로 stream으로 받아와서 local copy를 업데이트 할수 있음
- table-table join
  - 실시간으로 변경되고 있는 두개의 테이블 스트림들간의 조인
- time-dependence of joins
  - 두개의 stream에 있는 이벤트들을 대상으로 join을 수행할때, 특정 이벤트가 어느 이벤트와 조인해야 하는지 명확하지 않을때가 있다. 이 경우는 이벤트마다 unique id를 부여하고, unique id가 같은 이벤트들끼리 조인하도록 할수 있다.



## Fault Tolerance

- Stream processing에서 exactly once를 보장하기 위한 방법
  - Microbatching
    - Spark Streaming에서 사용되는 방법으로, batch size와 동일한 크기의 tumbling window를 제공
  - checkpointing
    - Apache Flink에서 사용되는 방법으로, 주기적으로 state의 checkpoint를 생성하고 결과를 durable storage에 기록한다.
    - 앱이 죽으면, 마지막 checkpoint부터 다시 처리할수 있다.
  - 하지만 위의 두가지 방법 모두 output이 stream processor를 떠나면 다시 회수할수 없다는 문제점이 있다.
  - 이러한 문제를 해결하기 atomic commit 기능을 도입할수 있다. 대표적으로 Google Cloud Dataflow, VoltDB, Apache Kafka(아마 카프카 트랜젝션을 말하는듯)에서 사용되고 있다.
  - 또한 write작업이 idempotent하도록 해서 exactly once를 보장할수도 있다. 이 경우 atomic commit기능보다 오버헤드가 적게든다.
- failure 발생시 상태 복구
  - 외부 시스템에 상태를 주기적으로 저장해놓고, 실패시 복구
  - 로그를 다시 읽어서 상태를 복구
  - 기반 인프라의 network bandwidth와 disk bandwidth 간의 trade-off에 따라 위의 두가지 방식의 퍼포먼스가 달라짐.

