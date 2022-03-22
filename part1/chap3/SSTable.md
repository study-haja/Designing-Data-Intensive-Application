## SSTable

key-value pair들을 key로 정렬된 상태로 저장하는 방식이다.

### 장점

1. merge과정이 간단하고 효율적이다. 여러 segment 파일들을 merge할때도 key값으로 정렬된 상태로 저장할 수 있다.
2. 파일에서 특정 key를 탐색하기 위해서, in-memory hashmap에 모든 key값을 인덱싱하지 않아도 된다. 일부만 저장해두고 key가 정렬되어있다는 특성을 활용할 수 있다.
3. 여러 레코드를 하나의 블락으로 압축하여 디스크에 저장할 수 있는데, 이 경우 데이터를 압축된 블락단위로 읽어 들임으로써 속도를 향상시킬수 있다. 그리고 디스크 공간도 절약된다.



원래 이러한 인덱싱 구조는 Patrick O'Neil이란 사람이 고안한 Log-Structured Merge-Tree(LSM-Tree)에서 설명된 적이 있다. merging and compacting sorted files 방식에 원칙을 둔 스토리지 엔진을 LSM storage engine이라고 한다.



Lucene에서 SSTable-like sorted files들을 사용하는것으로 알려져있다.



>  참고 : [LSM과 SSTable의 차이점](https://sjo200.tistory.com/56)



### Performance optimizations

- Bloom filter라는 memory-efficient data structure를 사용해서, key가 DB에 있는지 예측할 수 있게 해준다. 따라서 불필요한 디스크 읽기를 줄일 수 있다.
- compaction 조건과 타이밍을 정할 수 있다.
  - size-tiered compaction 
  - leveled compaction
