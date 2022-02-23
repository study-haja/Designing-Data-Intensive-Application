## Reliability

시스템에 fault가 발생하더라도 계속해서 동작하는 성질

> Fault vs Failure
>
> Fault란 보통 시스템의 단일 컴포넌트가 예상 spec에서 벗어나 있다는것을 의미하고,
>
> failure란 시스템이 유저에게 서비스를 제공하지 못하는 때를 말한다.

이제 fault의 유형에 대해서 알아보자.

### Hardware Faults

하드 디스크, RAM, 정전등 하드웨어에서 발생할수 있는 문제들을 말한다.

관리해야하는 하드웨어 머신의 수가 작을때는, 하드웨어 컴포넌트 이중화를 통해서 예방할수 있었지만 요즘처럼 수많은 머신에서 어플리케이션을 실행하는 경우에는 컴포넌트 이중화를 기법을 현실적으로 사용하기 어렵다. 요즘은 `software fault-tolerance technique` 이 더 선호된다.

### Software Errors

소위 말하는 소프트웨어 버그로 인해서 발생하는 fault를 말한다.

### Human Errors

시스템을 운영하는 사람들이 실수하는 경우 발생하는 fault를 말한다. 예를들어, 소프트웨어 배포시에 configuration값을 잘못 전달하여 에러가 발생하는 경우를 들수 있다.
