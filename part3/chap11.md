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