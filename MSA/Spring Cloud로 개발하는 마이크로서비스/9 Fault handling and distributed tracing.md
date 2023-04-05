마이크로서비스라는 것은 하나의 단일 어플리케이션이 아니라 여러개의 서비스로 개발되나 보니
각각의 서비스에 문제가 생겼을 때, 어떻게 처리를 해야하는지 해당하는 마이크로서비스의 시작점이 어디고
끝났을 때 반환값을 누구에게 줘야할지.

## CircuitBreaker

문제가 생겼던 서비스를 더이상 사용하지 않도록 막아주고 문제가 생겼던 서비스가 정상적으로 복구가 된다고 하면,
이전에 사용했던 것 처럼 정상적인 흐름으로 바꿔주는 장치

- 장애가 발생하는 서비스에 반복적인 호출이 되지 못하게 차단
- 특정 서비스가 정상적으로 동작하지 않을 경우 다른 기능으로 대체 수행 -> 장애 회피

Circuit Breaker는 2가지 경우가 있다.

- Open
  : MicroService에 문제가 있어 해당 서비스를 사용할 수 없음.
  -> 장애 회피(다른 기능으로 대체 수행)

- Closed
  : 정상적으로 다른 MicroService를 사용할 수 있다.

2019년도 까지는 Spring Cloud Bestflix Hystrix 라이브러리를 통해 CircuitBreaker를 구현을 했으나
2019년도 이후로는 더이상 개발하지 않고 유지보수만 하고 있는 상황인지라
Resilience4j라는 라이브러리로 대체해서 사용한다.

#### Resilience4j

- circuitbreaker
- ratelimiter
- bulkhead
- retry
- timelimiter
- cache

경량으로 Netflix의 Hystrix를 기반으로 해서 fault tolerance 작업을할 수 있다.
fault tolenrance?
: 에러가 발생을 했을때, 에러가 발생을 한다고 하더라도 정상적인
정상적인 서비스 처럼 가용할 수 있는 라이브러리다.
