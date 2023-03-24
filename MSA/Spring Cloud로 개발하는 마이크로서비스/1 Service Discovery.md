하나의 ip 밖에 없다면?
port 분산

가용할 수 있는 pc가 1대 이상?
server IP1:같은포트
server IP2:같은포트
server IP3:같은포트

### Eureka

: 서비스 discovery
외부의 서비스들이 마이크로서비스를 검색하기 위해서 사용되는 개념

일종의 전화부책

Key, Value가 저장되있다고 가정
어떠한 서버가 어느 위치에 있는지 등록하고 있는 정보가 Service Discovery

서비스들의 등록, 검색에 관련된 작업을 해주는 것을 Discovery Service라고 함.
-> Netflix의 Eureka라는 서비스를 사용해서 구현

Netflix의 클라우드 기술을 자바스프링 제단에 기부해서 사용할 수 있는 것이 Eureka 서비스

각각의 마이크로서비스가 전부다 자신의 위치정보를
스프링 Netflix Eureka 서버에 등록이라는 작업을 먼저한다.

마이크로서비스를 사용하고 싶은 클라이언트는 제일 먼저 자신이 필요한 요청 정보를 Load Balancer라던가 Gateway(API Gateway)
에다가 자신이 필요한 요청정보를 전달하게 되면 그 요청 정보가
Service Discovery에 전달이 되서 내가 필요한 정보가 어디에 있냐고 물어봄 -> Service Discovery는 이 요청에 해당 필요 정보가 저장된 서버로 가라고 시켜주게 되면 사용자 요청 정보가 해당 서버로 호출되고 그거에 대한 결과값을 가져간다.

ServiceDiscovery 해주는 역할은 각각의 마이크로서비스가 어디에 누가 저장되어 있으며, 요청정보가 들어왔을 떄 요청정보에 따라서
필요한 서비스들의 위치를 알려주는 역할을 한다.

사용자, 상품, 주문을 가지고 있는 간단한 온라인 쇼핑몰을 만들 예정
서비스에서 필요한 DiscoveryService로서 프로젝트 이름을 EcommerceDiscoveryService라고 지을 것임

DiscoveryService
application.yml

```yml
server:
  port: 8761

spring:
  application:
    name: discoveryservice

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

```yml
server:
  port: 8761
```

- 서버 포트 번호
- Eureka 서버가 WebService의 성격으로써 기동이 됨에 있어서 port번호를 어떻게 할지

```yml
spring:
  application:
    name: discoveryservice
```

- spring boot라는 프레임워크에서 마이크로서비스의 고유한 id 역할

```yml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

Eureka 라이브러리가 포함된 채 스프링 부트가 기동이 되면 기본적으로 Eureka Client 역할로써 어딘가에다 등록하는 작업을 시도하게 됨 그중에서 register-with-eureka, fetch-registry는 기본적으로 현재 작업하고 있는 것을 client 역할로 전화번호부에 등록하듯이 자신의 정보를 자신에게 등록하게 되는데 의미가 없는 작업이기 때문에 false라고 지정 Eureka 서버는 기동은 하되 자기 자신의 정보를 외부에 있는 다른 마이크로서비스가 Eureka 서버로 부터 어떤 정보를 주고받는 역할을 할 필요가 없기 때문에 자기 자신은 등록하지 않는 것!
-> 서버로써 기동만 되어 있으면 됨

#### 실행화면

- http://localhost:8761
  를 가보면 연결된 instance들을 볼 수 있다.

```yml
server:
  port: 9001

spring:
  application:
    name: user-service

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka
```

UserService
application.yml

```yml
service-url:
  defaultZone: http://127.0.0.1:8761/eureka
```

지금 만들고 있는 userservice는 eureka라는 서버에 등록된
eureka client 역할을 한다.
서버의 위치가 어디인지 그 항목을 지정하는 부분

### UserServiceApplication-2 생성

UserServiceApplication 뿐만 아니라 새로운 사용자 서비스를 등록하고자 함
Edit Configurations에서 UserServiceApplication-2를 생성
Environment의 VM options -Dserver.port=9002로 포트 번호 지정
이런 방법은 서버 자체의 코드가 변경되는 것이 아니기 때문에 매번 실행할 때 마다
이런 포트 값을 달리해서 제공할 수 있다.

- -D 옵션은 옵션을 추가하겠다는 의미
- server.port 라는 환경 설정(application.yml)값

서버 자체에 코드가 변경되는 것이 아니기 때문에 매번 실행할 때 마다 포트값을 달리할 수 있다.
한번 작성되어진 코드가 다시 빌드되고 다시 배포되어지는 과정이 아니라 서버를 기동하는 방법에 의해서
서버를 기동할 때 부가적인 파라미터를 전달 함으로써 동적으로 변경될 수 있는 서버 포트를 지정할 수 있다는 특징

Eureka Service Directory안에 UserService가 2개가 등록된걸 확인할 수 있다.
외부에서 클라이언트 요청이 UserService에 전달이된다라고 하면 이제 discovery service 안에서
9001으로 전달될지 9002번으로 전달될지 어떠한 인스턴스가 살아있는지 정보값을 Gateway라던가
라우팅 서비스한테 전달해주게 되면 2가지 서비스에 대해서 분산된 서비스가 실행되게 할 수 있게 되었다.

### Gradle 빌드 도구로 개발 된 애플리케이션에 포트 번호를 변경

1.  gradle build 명령어 실행 후, java -jar "빌드 된 jar 파일" --server.port=8081 (또는 java -jar -Dserver.port=8081 "빌드 된 jar 파일"

2.  gralde 명령어 실행 시 설정할 옵션(예, 포트번호)를 같이 지정할 수 있다. 이때는 build.gradle 설정 파일에 아래와 같은 설정을 추가한 다음에 실행면 된다.

```groovy
bootRun {
    if (project.hasProperty('args')) {
        args project.args.split(',')
    }
}
```

```
$  gradle bootRun -Pargs="--server.port=8081"
```

application.yml

```yml
server:
  port: 0
```

랜덤 포트를 사용하겠다는 의미, 같은 서비스를 사용하더라도 매번 실행할 때마다 랜덤한 포트 번호가 할당

```yml
eureka:
  instance:
    instanc-id: ${spring.cloud.client.hostname}:${spring.application.instance_id:${random.value}}
```

- Spring Cloud의 서비스 디스커버리를 사용하는 애플리케이션에서 사용될 수 있는 인스턴스 ID 생성하는데 사용
- ${spring.cloud.client.hostname}: 해당 인스턴스가 실행되는 호스트의 이름을 가져오는 Spring 환경 속성, 서비스 디스커버리를 사용하여 애플리케이션 인스턴스를 식별하는데 사용
- ${spring.application.instance_id:${random.value}}: 인스턴스의 고유 ID 생성
