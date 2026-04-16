# WebClient vs OpenFeign

### 1. WebClient (Spring WebFlux)

> Spring 5부터 도입된 비동기, 논블로킹(Non-blocking) 방식의 리액티브 웹 클라이언트

동작 방식: 논블로킹(Non-blocking) 방식을 사용하여, 요청을 보낸 후 응답을 기다리지 않고 다른 작업을 수행할 수 있음

**장점**
- 적은 리소스로 대량의 동시 요청을 효율적으로 처리
- 동기(Blocking)와 비동기 방식을 모두 지원
- RestTemplate의 공식적인 대안으로 권장

**단점**
- 리액티브 프로그래밍(Project Reactor)에 대한 학습 곡선이 높음
- 연산자 지옥 (Operator Hell): 비즈니스 로직이 고도화될수록 가독성이 떨어지고 디버깅이 어려워짐
- 스택 트레이스(Stack Trace)의 파편화: 디버깅 어려움
- 사이드 이펙트 관리의 어려움: 로그 등을 위해 중간에 값을 가로채기 어려움

**해결책 / 대안**

Kotlin Coroutines: suspend 함수와 Flow를 사용하여 비동기 코드를 동기 코드처럼 쓸 수 있음

Java Virtual Threads: MVC 모델에 가상 스레드를 도입하여 높은 동시성을 확보

### 2. OpenFeign (Spring Cloud OpenFeign)

> OpenFeign은 선언적(Declarative) 웹 서비스 클라이언트
>
> 인터페이스를 작성하고 어노테이션을 붙이는 것만으로도 HTTP 요청을 보낼 수 있음

동작 방식: 기본적으로 동기(Blocking) 방식

**장점**
- 코드가 매우 간결하고 가독성이 뛰어남
- Spring MVC 어노테이션(@GetMapping, @PostMapping 등)을 그대로 사용할 수 있음
- Spring Cloud 생태계(Load Balancer, Circuit Breaker 등)와 통합이 쉬움

**단점**
- 실행 시점의 동적인 설정 변경이 WebClient에 비해 다소 까다로울 수 있음

### 3. 비교 요약

| 특징 | WebClient | OpenFeign |
| :--- | :--- | :--- |
| **방식** | 비동기 / 논블로킹 (Reactive) | 동기 / 블로킹 (Blocking) |
| **복잡도** | 상대적으로 높음 (Flux/Mono) | 낮음 (인터페이스 기반) |`spring-cloud-starter-openfeign` |
| **성능** | 고부하, 대량의 동시 접속에 유리 | 일반적인 MSA 환경에 충분 |
| **스타일** | 함수형 스타일 | 선언적 스타일 |

### 선택 기준
OpenFeign을 추천하는 경우:
1. 리액티브 프로그래밍에 익숙하지 않을 때
2. 간결한 코드와 빠른 개발 속도가 중요할 때
3. Spring MVC 기반의 서블릿 애플리케이션을 개발할 때

WebClient를 추천하는 경우:
1. Spring WebFlux를 사용하는 리액티브 스택일 때
2. 초당 요청 수가 매우 많아 성능 최적화가 필수적일 때
3. 단일 요청 내에서 여러 외부 API를 병렬로 호출하고 합쳐야 할 때