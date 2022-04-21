# 분산 데이터

여러대의 머신으로 디비를 분산하고싶은 요인은 아래와같은 장점이 있기 때문일것이다.

- Scalability
- Fault tolerance/high availability
- Latency



여전히 머신의 스펙을 높여서(CPU수 증가, 메모리 칩 또는 디스크 수 늘리기) 더많은 부하를 견디게 할수는 있다. 이런 구조를 **shard-memory architecture** 기반이라고 한다.

하지만 다음 문제가 있다.

- 비용이 급격하게 증가한다.
- 보틀낵때문에 이러한 하드웨어 컴포넌트들을 늘린다고 해서 정량적으로 처리량이 증가하지 않는다.
- 여러 하드웨어 컴포넌트들을 전원을 끌필요없이 교체할수는 있지만 하나의 지리적 위치에서만 가능하다.

shared-disk architecture의 경우에 CPU와 RAM은 독립적으로 두고 디스크를 독립적으로 둘수 있지만 locking 오버헤드 때문에 scalable하지 않다.