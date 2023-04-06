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

## Microservice 분산 추적

### Zipkin

- Twitter에서 사용하는 분산 환경의 Timing 데이터 수집, 추적 시스템 (오픈소스)
- Google Drapper에서 발전하였으며, 분산환경에서의 시스템 병목 현상 파악
- Collector, Query Service, Databasem WebUI로 구성
- Sapn
  - 하나의 요청에 사용되는 작업의 단위
  - 64bit unique ID
- Trace
  - 트리 구조로 이뤄진 Span(여러개) 셋
  - 하나의 요청에 대한 같은 Trace ID 발급

#### Spring Cloud Sleuth

- 스프링 부트 애플리케이션을 Zipkin과 연동
- 요청 값에 따른 Trace ID, Span ID 부여
- Trace와 Span Ids를 로그에 추가 가능

  - serlet filter
  - rest template
  - scheduled actions
  - message channels
  - feign client

- Zipkin 다운로드 명령어

```
curl -sSL https://zipkin.io/quickstart.sh | bash -s
```

- Zipkin 실행 명령어

```
java -jar zipkin.jar
```

- Zipkin 접속

```
http://localhost:9411/zipkin/
```

- Zipkin dependency 추가(오류로 2.2.3.RELEASE 버전 사용)

```
// https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-zipkin
	implementation group: 'org.springframework.cloud', name: 'spring-cloud-starter-zipkin', version: '2.2.3.RELEASE'
```
