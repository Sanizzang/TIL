각각의 마이크로서비스가 가져야 될 구성 정보 파일
우리는 구성 정보 파일로써 application.yml 파일을 사용했었는데,
yml 파일의 내용이 변경이되버리면 어플리케이션 자체가 다시 빌드가 되고 배포가 되어야 된다.
그런 방법을 개선하기 위해서 어플리케이션 내부에서 구성 파일을 가지고 있는 것이 아니라 외부에 있는 시스템을 통해서 구성 파일 정보를 관리할 수 있는 기능

### Spring Cloud Config

- 분산 시스템에서 서버, 클라이언트 구성에 필요한 설정 정보(application.yml)를 외부 시스템에서 관리
- 하나의 중앙화 된 저장소에서 구성요소 관리 가능
- 각 서비스를 다시 빌드하지 않고, 바로 적용 가능
- 애플리케이션 배포 파이프라인을 통해 DEV-UAT-PROD 환경에 맞는 구성 정보 사용

git-local-repo에 ecommerce.yml 파일 생성

```yml
token:
  expiration_time: 86400000
  secret: user_token

gateway:
  ip: 192.168.35.233
```

### Config-Service 프로젝트 생성

application.yml 설정

```yml
server: port:8888

spring:
  application:
    name: config-service
  cloud:
    config:
      server:
        git:
          uri: file:///D:\\Study\\git-local-repo
```

#### UserMicroservice 코드 추가

1. Dependencies 추가

- spirng cloud starter config
- spring cloud starter bootstrap

2. bootstrap.yml 추가

현재 application.yml은 각종 설정정보가 여기에 들어가 있다. port정보, spring cloud의 어플리케이션 name이라던가 아니면 접속하고자하는 외부에 있는 api 주소 정보라던가 아니면 라우팅 정보라던가 이러한 정보들을 어플리케이션의 상황에 맞춰서 정의를 해왔다.
여기에 bootstrap.yml을 만들고자 한다.
application.yml보다 우선순위가 높은 설정파일이 하나 생기는 것이다.

```yml
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
```

bootstrap.yml에는 우리가 읽어 오고자하는 configuration 정보의 위치를 저장하고자 한다.

물론 이 정보는 application.yml 안에 내부적으로 같이 가지고 있어도 되는 부분이다. 하지만 우리가 하고자 하는 작업의 핵심은 원래 가지고 있었던 application.yml의 특정한 부분을 띄어서 별도의 다른 공용 서버 역할을 해주는 spring cloud config 서버를 이용해보겠다는 이번 프로젝트의 목적이다.
해당하는 부분이 읽혀지는 부분들을 앞에 있는 application.yml 파일 보다는 먼저 작업을 해야지만 전체적으로 우리가 필요했던 resource가 다 맞아 떨어진다.
따라서 오른쪽에있는 cloud config에 대한 정보를 등록해 줄 수 있는 파일이 필요하다.

#### Configuration 정보가 변경된다면?

- 서버 재기동
  서버를 매번 재기동하는 방법은 좋은 방법은 아니다.

- Actuator refresh
  현재 User라는 MicroService를 재부팅하지 않은 상태에서도 우리들이 필요한 정보를 얻어올 수 있다.

  - Application의 상태라던가 모니터링
  - Metric 수집을 위한 Http End point 제공

```yml
# Actuator 구성을 지정하는데 사용되는 부분
management:
  # Actuator 엔드포인트를 구성하는 데 사용되는 부분
  endpoint:
    # Actuator 엔드포인트가 웹 요청에 응답하는 데 사용되는 부분
    web:
      # Actuator 엔드포인트에서 노출할 엔드포인트를 구성하는 데 사용되는 부분
      exposure:
        # Actuator 엔드포인트에서 노출할 엔드포인트의 목록을 구성하는 데 사용되는 속성
        include: refresh, health, beans
```

api-gateway-service에 httptrace 빈 추가

```java
    // HTTP 요청과 응답에 대한 추적 정보를 저장하고 관리하는 이넡페이스
	@Bean
	public HttpTraceRepository httpTraceRepository() {
        // 메모리에 HTTP 추적 정보를 저장함
		return new InMemoryHttpTraceRepository();
	}
```

configuration 값을 변경하고
/actuator/refresh에 send하면 반영이 된다.

- Spring cloud bus 사용
  클라우드 버스를 사용하게 되면 2번째 방법보다 효율적으로 정보를 사용할 수 있다.

#### Profiles 사용한 COnfiguration 적용

git-local-repo에 ecommerce-dev.yml, ecommerce-prod.yml 파일 추가

user-service, api-gateway-service의 bootstrap.yml 파일에 profiles 설정 추가

```yml
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
  profiles:
    active: dev
```

#### configuration 값 git에 등록

- ecommerce configuration 정보들을 git remote 저장소에 올린다.

config-service의 application.yml 파일 수정

```yml
server: port:8888

spring:
  application:
    name: config-service
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Sanizzang/spring-cloud-config
          # Private으로 설정시 username, password 정보도 입력해야함
        #   username: [your username]
        #   password: [your password]
```

#### Native File Repository에서 설정 값 불러오기

```yml
spring:
  application:
    name: config-service
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: file:///${user.home}\\file경로
        git:
          uri: https://github.com/Sanizzang/spring-cloud-config
```

## Spring Cloud Bus 사용하기

- 분산 시스템의 노드를 경량 메시지 브로커와 연결
- 상태 및 구성에 대한 변경 사항을 연결된 노드에게 전달(Broadcast)

노드: MicroService
메시지 브로커: RabbitMQ 사용 예정

Spring Cloud Config Server + Spring Cloud Bus
데이터가 갱신되었음을 알려줄 예정

AMQP(Advanced Message Queing Protocol)을 이용해서
데이터가 변경되었음을 알려줄 예정

- AMQP: 메시지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜
  - 메시지 지향, 큐잉, 라우팅(P2P, Publisher-Subscriber), 신뢰성, 보안
  - Erlang, RabbitMQ에서 사용
    - Erlang: 함수형 병행성 프로그래밍 언어, 가벼운 프로세서
- Kafka 프로젝트
  - Apache Software Foundation이 Scalar 언어로 개발한 오픈 소스 메시지 브로커 프로젝트
  - 분산형 스트리밍 플랫폼
  - 대용량의 데이터를 처리 가능한 메시징 시스템

#### RabbitMQ

- 메시지 브로커
- 초당 20+ 메시지를 소비자에게 전달
- 메시지 전달 보장, 시스템 간 메시지 전달
- 브로커, 소비자 중심

#### Kafka

- 초당 100k+ 이상의 이벤트 처리
- Pub/Sub, Topic에 메시지 전달
- Ack를 기다리지 않고 전달 가능
- 생산자 중심

Kafka가 대용량의 데이터를 처리하고자 할 때 많이 선택되는 솔루션이고 RabbitMQ는 더 적은 데이터를 안전하게 전달하는 것을 보장시켜주는 데이터라고 보면된다.

Spring Cloud Bus는 연결되어진 노드에게 전달하는 방송(Broadcast)역할을 한다.

RabbitMQ라는 메시지 큐잉 서비스에 configuration 서버, api gateway 서버, usermicro 서버가 연결되어 있는 구조

Spring Cloud Config는 구성 정보를 중앙 집중식으로 저장하고, 클라이언트 애플리케이션은 해당 서비스에서 구성 정보를 로드하여 사용할 수 있다.
이를 통해 분산 시스템의 다양한 애플리케이션에서 중복되는 구성 정보를 제거하고, 애플리케이션의 유지 보수성을 높이는 데 도움이 된다.

RabbitMQ는 분산 메시징 시스템으로서, 애플리케이션 간에 비동기적으로 메시지를 전송할 수 있도록 도와준다. RabbitMQ를 사용하면 애플리케이션 간의 결합도를 낮추고, 느슨한 결합을 유지할 수 있어 애플리케이션의 확장성과 유연성을 높일 수 있다.

Spring Cloud Config와 RabbitMQ를 함께 사용할 경우, RabbitMQ를 사용하여 구성 정보 변경 사항을 다른 애플리케이션에게 전파할 수 있다. 예를 들어, 구성 서비스에서 구성 정보를 업데이트하면 해당 변경 사항을 RabbitMQ에 게시하고, 다른 애플리케이션은 해당 변경 사항을 구독하여 자동으로 업데이트된 구성 정보를 수신할 수 있다. 이를 통해 애플리케이션 간이 구성 정보 변경 사항을 쉽게 전파하고, 애플리케이션의 유연성과 확장성을 높일 수 있다.
