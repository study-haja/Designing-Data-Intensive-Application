## SQL

- 가장 잘 알려진 Data Model은 SQL이며, 1970년 Edgar Codd에 의해 제안되었다. 
- relational model : data는 relations으로 구성되있고, 각 relation은 tuple들로 구성되어있다.



### The Birth of NoSQL

- NoSQL을 선택하게 된 배경
  - relational database보다 scalability를 더 쉽게 달성가능(large dataset, high write throughput)
  - open source software 기반이라 경제적
  - relational model에는 지원하지 않는 query operation 지원
  - more dinamic and expressive data model



### Object-Relation Mismatch

- JSON으로 데이터를 표현하면 locality가 더 좋다. 반면 multi-table schema로 표현하면 데이터를 fetch할때 여러 테이블들을 조인해야 한다.