## Maintainability

유지보수성을 좋게 하기 위해서는 다음의 디자인 원칙을 눈여겨 봐야 한다.

- Operability : Ops team이 시스템을 매끄럽게 실행할수 있게하기
- Simplicity : 시스템의 복잡도를 최소화 하기
- Evolvability : 시스템 확장이 쉽도록 하기



### Operability : Making Life Easy for Operations

운영을 자동화 시킬수 있는 부분은 자동화 시키고, 수동으로 배포해야 하는경우 배포 과정자체를 쉽게 만들기



### Simplicity

- `accidental complexity` 를 제거해야 한다. `accidental complexity`란 소프트웨어가 풀고자 하는 문제로 부터 기인한 복잡성이 아닌 구현자체로 발생한 복잡성이다.
- 추상화된 High Level Language or SQL을 사용한다. SQL은 on-disk in-memory data structure 등을 숨겨주는 추상화 툴이라고 할수 있다.



### Evolvability

- Agile working pattern은 변화에 적응하기 위한 프레임워크를 제공하는데, TDD와 refactoring이 대표적인 예이다.