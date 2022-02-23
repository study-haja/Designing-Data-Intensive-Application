## Scalability

>system's ability to cope with increased load



### Describing performance

- performance 측정에 mean은 좋은 방법이 아니다. 그보다는 median(50th percentile, p50)이 좋은 방법이다. 50% 정도는 해당 값보다 적은 응답시간을 경험하고 나머지 50%정도는 해당 값보다 더 긴 응답시간을 경험한다고 이해할 수 있다.



### Approches for Coping with Load

- Scaling up(vertical scaling) : 구조가 더 간단하지만 high-end machine으로 업그레이드 하는것은 비용이 많이 들어 결국 scaling out을 피할 수 없다.
- Scaling out(horizontal scaling) : shard-nothing architecture라고도 부르며, 값싼 장비 여러대로 로드를 분산할 수 있다는 장점이 있다.
- 실용적인 접근은 위 두가지 접근을 혼합하는 것이다. 예를들어 적당히 powerful machine 여러대를 가지고 운영하는 것이다.
- elastic system이란 자동으로 컴퓨팅 리소스를 추가할 수 있다는 것을 의미한다. load를 예측하기가 어려운 경우 elastic 하면 좋지만 elastic 하지 않는 시스템은 더 단순하고 운영적으로 더 안정적일 수 있다.