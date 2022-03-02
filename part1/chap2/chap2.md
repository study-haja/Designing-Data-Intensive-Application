## SQL

- 가장 잘 알려진 Data Model은 SQL이며, 1970년 Edgar Codd에 의해 제안되었다. 
- relational model : data는 relations으로 구성되있고, 각 relation은 tuple들로 구성되어있다.



### The Birth of NoSQL

- NoSQL을 선택하게 된 배경
  - relational database보다 scalability를 더 쉽게 달성가능(large dataset, high write throughput)
  - open source software 기반이라 경제적
  - relational model에는 지원하지 않는 query operation 지원
  - more dynamic and expressive data model



### Object-Relation Mismatch

- JSON으로 데이터를 표현하면 locality가 더 좋다. 반면 multi-table schema로 표현하면 데이터를 fetch할때 여러 테이블들을 조인해야 한다.



### Many-to-One and Many-to-Many Relationships

- Document model은 one-to-many 관계의 데이터를 표현하는데는 적합하지만, many-to-many 관계의 데이터를 표현하는데는 적합하지 않다. join을 지원하지 않는 데이터베이스에서도 표현할 수는 있지만 application단의 코드가 복잡해진다.



### Network model vs Relational model

- Network Model
  - `Conference on Data Systems Languages(CODASYL)` 커미티에 의해서 표준화되었음.
  - 레코드들 간의 연결은 foreign key를 통해서 이루어지는 것이 아니라, 프로그래밍 언어의 포인터와 유사한 형태로 이루어진다.
  - 1970년대의 limited hardware 성능에서는 효율적인 솔루션이었지만, DB를 쿼리하고 업데이트하는 과정을 코드로 표현해야 했으므로 complicated, inflexible 하다. 가령, 데이터를 탐색해나가는 path가 수정된다면 원래 해당 데이터를 탐색하는 코드의 path를 변경된 path로 모두 수정해야 한다.
- Relational Model
  - query optimizer가 자동으로 어떤 쿼리를 어떤 순서대로 어떤 인덱스를 사용할지를 결정하므로, 데이터 탐색로직을 신경쓸 필요가 없다.
- Network model vs Document model
  - document model에서는 many-to-many 관계를 표현할때 다른 document를 가리키는 document reference를 사용하는데 이는 relational model에서의 foreign key와 유사한 개념이다. 따라서 network model과는 차이가 있다.



### Relational vs Document DB

- Document DB는 일반적으로 스키마가 유연하다는 장점이 있다. 하지만 다른 테이블과 조인을 해야할때 조인을 지원하지 않는 document DB에서는 조인로직을 어플리케이션에 구현해야하고 이는 코드의 복잡성을 증가시키고, 성능이 감소되는 원인이 된다.
- 스키마가 유연한것이 도움이 되는 상황은 두가지가 있다.
  - 첫번째는 테이블당 오브젝트 타입을 독립적으로 저장하는것이 실용적이지 않을때(즉, 하나의 테이블에 모든 오브젝트 타입을 저장하는것이 더 좋을때) 
  - 두번째는 데이터 구조가 외부시스템에 의해서 결정이 되어, 스키마를 통제할수 없을때
- Document DB는 document를 읽는 시점에, 일부분만을 읽을수는 없기 때문에 document의 크기가 매우 큰경우는 비효율적일수 있다. 그리고 하나의 document에서 일부분만 업데이트 하고자하는 경우도 전체 document를 다시 쓰는 작업을 해야하기 때문에 비효율적이다.
- data locality는 column-family concept에서도 사용이 되고 있다.
- 관계형 DB에서도 JSON document 타입을 지원하기 시작하면서, relational DB와 document DB가 하나로 수렴되고 있는것으로 보인다.



### Query Language for Data

- Imperative 방식 : Programming language를 사용해서 쿼리로직을 표현

  - ```javascript
    function getSharks() {
      var sharks = [];
      for (var i = 0; i < animals.length; i++) {
        if (animals[i].family == "Sharks") {
    	    sharks.push(animals[i]);      
        }
      }
      return sharks;
    }
    ```

  - MapReduce

- Declarative 방식 : SQL 사용

  - ```sql
    SELECT * FROM animals WHERE family = 'Sharks';
    ```

  - 내부 구현이 명시되어 있지 않으므로, DB 단에서 최적화를 자동으로 수행할 수 있는 부분이 많다.

  

### Graph-Like Data Models

- 어플리케이션이 one-to-many relationship을 주로 가지고 있다면 document model이면 충분하다. many-to-many 관계가 있어도 그리 복잡하지 않다면 relational model이면 충분하다. 하지만 데이터의 관계가 점점 복잡해질수록 graph model을 고려해보는것이 좋다.



### Graph DB vs Network model

Graph DB는 앞서 소개한 CODASYL과는 4가지 중요한 차이점이 있다.

1. CODASYL은 특정 레코드 타입이 포함할수 있는 레코드 타입을 스키마로 정의하는 반면, Graph DB에서는 하나의 vertex는 edge를 통해서 어떠한 vertex와도 연결이 가능하다.
2. CODASYL에서 특정 레코드에 접근하는 방법은 access path를 순회하는 방법 뿐인 반면, Graph DB에서는 unique id로 바로 특정 vertex에 접근가능하며 특정 값을 가진 vertex들을 찾기 위해서 인덱스를 사용할수도 있다.
3. CODASYL에서 레코드의 자식을 표현할때 항상 순서를 고려해서 저장해야 하는 반면, Graph DB에서는 vertex들 간의 순서를 고려할 필요가 없다.
4. CODASYL에서는 모든 쿼리가 imperative 한 반면, Graph DB는 imperative, declarative 쿼리 모두 지원한다.



### Summary

1. Data locality가 중요하고 Schema가 유연해야 되며, many-to-many 관계가 없는 구조라면 document model을 선택
2. many-to-many 관계가 존재하고 simple하면 relational model
3. many-to-many 관계가 존재하고 complex하면 graph model
