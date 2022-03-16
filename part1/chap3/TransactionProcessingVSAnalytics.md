## Transaction Processing vs Analytics

- 1980년대 후반부터 1990년대 초반, analytics를 위해서 독립된 DB를 운영하기 시작했는데 이러한 DB를 data warehouse라고 불렀다.



### Data Warehousing

보통 OLTP 시스템을 운영하는 DBA들은 데이터 분석가들이 ad hoc analytic query를 OLTP database에 실행하는걸 원치않는다. 왜냐하면 동시에 실행중인 트랜젝션의 성능에 영향을 주기 때문이다.

데이터의 크기가 작은 경우는 굳이 독립된 DB를 운영할 필요가 없지만, 그렇지 않으면 독립적인 DB를 사용하는것이 좋다.

data warehouse(OLAP system)에서 analytic query를 처리하는데 있어서 indexing 알고리즘은 별 효과가 없다.

많은 data warehouse vendor들이 시스템을 유료로 제공하고 있는데 반해, SQL-on-Hadoop 오픈소스 프로젝트들(Apache Hive, Spark SQL, Cloudera, Impala, Facebook Presto 등)이 존재한다.



### Stars and Snowflakes : Schemas for Analytics

많은 데이터 웨어하우스가 바로 star schema(또는 dimensional modeling)라는 꽤 formulaic style로 사용된다.

star schema라고 불리는 이유는 하나의 거대한 칼럼들을 가지는 fact table row에서 다른 여러 테이블(dimension table) row들을 참조하고 있는 모양자체가 별모양이기 때문이다. 

여기서 조금 변형된 형태가 Snowflake schema이다. Snowflake schema는 dimension들이 한번더 subdimension들로 나누어진다. Snowflake schema들은 star schema보다 더 정규화된 형태이지만 star schema가 분석가들이 더 작업하기 쉬우므로 선호된다.
