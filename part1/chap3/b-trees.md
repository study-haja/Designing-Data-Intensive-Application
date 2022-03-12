## B-Tree

### Comparing B-Trees and LSM-Trees

실제 경험상 LSM-tree들은 쓰기작업이 빠르고, B-Tree는 읽기 작업이 더 빠르다.



### Advantages of LSM-Trees

- higher write throughput than B-Trees : write amplification이 더 적고, 특히 hard drive를 사용할시 sequential write작업이 random access write 작업보다 빠르므로 장점이 더 부각된다.
- B-Tree 보다 더 작은 파일들을 생성하고, 더 압축이 잘되며 B-Tree처럼 fragmentation현상이 감소한다.



### LSM Tree의 단점

- 데이터가 많아질수록 파일 머지 과정에서 DISK bandwidth가 더 높아진다.
- 데이터가 저장되는 속도에 비해 compaction속도가 느리면 디스크 공간이 부족할 수 있다.
- 같은 키에 대한 데이터가 여러 장소에 저장될 수 있어서 weak transactional semantic 제공