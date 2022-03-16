## Hash Index

key value쌍을 파일에 쓰고나서 key가 저장되어있는 파일의 byte offset을 메모리의 hashmap에 업데이트 한다.

여기서 파일에 데이터를 계속 쓰다보면 디스크 공간이 부족해질 수 있으므로 compaction을 진행할수 있다.

>Compaction : key의 최근 value만 남기고 나머지는 제거하는 작업

compaction작업이 끝나면 예전버전의 segment 파일은 삭제하여 디스크 공간을 확보할 수 있다.

### 장점

- 데이터 쓰기 속도가 빠르다.



### 한계점

- hashmap의 사이즈가 커서 메모리에 올릴 수 없는 경우, on-disk hashmap을 사용해야 하는데 이 경우 random access I/O 성능이 좋지 않다. 
- Range query를 처리하려면, key별로 검색을 해서 데이터를 로드해야 하므로 b-tree index보다 성능이 떨어진다.

