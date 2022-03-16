## Column oriented storage

`column-oriented-storage`의 아이디어는 하나의 row에 있는 모든값을 함께 저장하는 것이 아니라 각각의 칼럼에 있는 값들을 함께 저장하는 것이다.



### Column Compression

- 칼럼에 있는 데이터에 따라서, 다른 compression이 가능한데, 예시로 bitmap encoding이 있다.

- column oriented storage layout은 CPU cycle을 효율적으로 만드는데 이점이 있다.



### Sort order

column oriented storage에서도 검색속도 증가를 위해서 데이터를 정렬해서 저장 할수 있고, 이는 데이터가 같은 데이터가 한곳에 모여있도록 할수 있으므로 압축률을 증가시킨다. 

이때 특정 칼럼 순서대로 정렬해서 저장할때, 칼럼 combination 여러개에 대해서 데이터를 정렬하여 데이터를 독립적으로 보관 할수 있다. 어차피 데이터를 복제해야 하므로, 복제할때 다른 칼럼들을 기준으로 데이터를 정렬해서 보관하는 것이다. row oriented store에서 여러 세컨더리 인덱스를 갖는것과 유사하지만 결정적인 차이점은 세컨더리 인덱스는 결국 heap file에 저장된 데이터에 대한 포인터를 가지고 있는 반면에, column oriented storage에서는 실제 데이터가 파일에 저장된다.



### Writing to Column-oriented Storage

앞서 다룬 압축과 정렬은 칼럼 기반 스토리지에 읽기작업을 빠르게 하지만 쓰기 작업은 더 어렵게 만든다. 이런 작업은 LSM tree로 해결이 가능하다. 인메모리 스토어에 데이터가 행기반이냐 열기반이냐 상관없이 충분히 적재되면 칼럼 파일들에 머지 시킬수 있다.



### Aggregation: Data Cubes and Materialized Views

aggregation(sum, count 등) 쿼리의 결과를 캐싱해서 속도를 개선할 수 있는데, 이러한 캐시를 만드는 방법중 하나가 materialized view이다. 관계형 데이터 모델에서는 종종 standard view로 정의가 되는데, 여기서 view란 어떤 쿼리의 결과를 갖고 있는 table-like 객체를 말한다.  여기서 materialized view와 standard view의 차이점은, materialized view는 실제 쿼리결과의 실제 카피본이지만 virtual view는 단지 writing 쿼리에 대한 shortcut이라는 점이다. 실제 virtual view를 읽을때, SQL 엔진이 뷰를 내부 쿼리로 확장한후 처리한다.

그밖에 materialized view의 특별한 경우인 data cube or OLAP cube도 있는데, 이 경우 특정 쿼리에 대해서 pre compute을 빠르게 할수 있다는 장점이 있다. 하지만 특정 쿼리에만 응답을 해준다는 한계점이 존재한다.

