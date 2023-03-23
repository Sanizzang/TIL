### API Gateway Service

사용자가 설정한 라우팅 설정에 따라서 각각 endpoint로 클라이언트 대신해서 요청하고 응답을 받으면 다시 클라이언트한테 전달해주는 proxy 역할을 하게 된다.
시스템의 내부 구조는 숨기고 외부의 요청에 대해서 적절히 가공해서 응답할 수 있다는 장점을 가지고 있음.

각각의 마이크로서비스로 요청되는 모든 정보에 대해서 일괄적으로 처리할 수 있게 됨

- 인증 및 권한 부여
- 서비스 검색 통합
- 응답 캐싱
- 정책, 회로 차단기 및 QoS 다시 시도
- 속도 제한
- 부하 분산
- 로깅, 추적, 상관 관계
- 헤더, 쿼리 문자열 및 청구 변환
- IP 허용 목록에 추가

## Netflix Ribbon

### Spring Cloud에서의 MSA간 통신

#### RestTemplate

RestTemplate은 전통적으로 하나의 웹 어플리케이션에서 다른 어플리케이션을 사용하기 위해 자주사용되었던 API

```java
RestTemplate restTemplate = new RestTemplate();
restTemplate.getForObject("http://localhost:8080/", User.class, 200);
```

#### Feign Client

```java
@FeignCLient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();
}
```

Spirng Cloud에서는 Feign Client를 통해 API를 호출할 수가 있는데,
인터페이스를 만들고 웹으로 호출하고 싶은 추가적인 마이크로 서비스를 등록
만약 User라는 서비스에서 FeignClient를 등록을 하고 store라는 서비스를 호출하겠다고 하면
굳이 직접적인 서버의 주소라던가 포트번호 없이 마이크로서비스 이름으로 호출할 수 있다.

### Ribbon: Client side Load Balancer

- 서비스 이름으로 호출
- Health Check

client가 직접적으로 microservice를 호출하는 방법은 좋지 않음
그래서 API Gateway를 중간에 놔야 하는데 그 작업을 별도의 서비스가 시스템을 구축하지 않고
클라이언트측 내부에 Ribbon이라는 서비스를 구축해서 사용하기 시작함
클라이언트 사이드에서 사용할 수 있는 Load Balancer임

- Spring Cloud Ribbon은 Spring Boot 2.4에서 Maintenance 상태

## Netflix Zuul

- Routing
- API gateway

현재 Spring boot 2.4 이상부터는 Netflix Zuul과 Ribbon은 지원하지 않기 때문에
Netflix Ribbon -> Spring Cloud Loadbalancer
Netflix Zuul -> Spring Cloud Gateway
로 대체해서 사용하는걸 권장한다

#### First Service, SecondService 구축

```java
@RestController
@RequestMapping("/")
public class FirstServiceController {

    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to the First service.";
    }

}
```

```yml
server:
  port: 8081

spring:
  application:
    name: my-first-service

eureka:
  client:
    register-with-eureka: false
    fetchRegistry: false
```

```java
@RestController
@RequestMapping("/")
public class SecondServiceController {

    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to the Second service.";
    }

}
```

```yml
server:
  port: 8082

spring:
  application:
    name: my-second-service

eureka:
  client:
    register-with-eureka: false
    fetchRegistry: false
```

Zuul Service

```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulServiceApplication.class, args);
    }
}
```

```yml
server:
  port: 8000

spring:
  application:
    name: my-zuul-service

zuul:
  routes:
    first-service:
      path: /first-service/**
      url: http://localhost:8081
    second-service:
      path: /second-service/**
      url: http//localhost:8082
```

ZuulFilter를 통한 사전(ex. 인증서비스), 사후(ex.로깅) 처리 가능

```java
@Slf4j
@Component
public class ZuulLoggingFilter extends ZuulFilter {
    @Override
    public Object run() throws ZuulException {
        log.info("**************** printing logs : ");

        // RequestContext를 통해 Servlet을 불러옴
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info("**************** " + request.getRequestURI());

        return null;
    }

    @Override
    public String filterType() {
        return "pre";
    }
}
```

## Spring Cloud Gateway 구축

```yml
server:
  port: 8000

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka

spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/**
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
```

Zuul에서는 동기방식으로 Tomcat 서버를 기동했었는데
Spring Cloud Gateway를 사용하면 비동기 방식으로(Netty) 작동이 된다.

Client가 First Service와 Second Service를 사용하고 싶다고 가정
그럼 중간에 Spring Cloud gateway가 자리를 잡고 있는다.
Client가 Spring Cloud gateway에 어떤 요청을 전달하게 되면 데이터서비스에서 First Service로 갈지 Second Service로 갈지 판단한 다음에 서비스에 요청을 분기를 해준다.

Spring Cloud에서 일어나는 작업을 조금 더 확장해서 보면
먼저 Gateway Handler Mapping을 통해 클라이언트로부터 어떤 요청이 들어왔는지 요청정보를 받고 Predicate를 통해 그 요청에 대한 사전요건 즉 어떤 이름으로 요청되었는지 조건을 분기해준다.

다음에 Pre Filter(사전 필터) Post FIlter(사후 필터)를 작성을 해서 요청 정보를 구성할 수 있다.

작업하는 방법은 Property로 작업할 수도 있고 Java Code로도 작업할 수 있다.

Spring Cloud Gateway 필터링

```java
@Configuration
public class FilterConfig {

    @Bean
    public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route(r -> r.path("/first-service/**")
                        .filters(f -> f.addRequestHeader("first-request", "first-request-header")
                                        .addResponseHeader("first-response", "first-response-header"))
                        .uri("http://localhost:8081"))
                .route(r -> r.path("/second-service/**")
                        .filters(f -> f.addRequestHeader("second-request", "second-request-header")
                                .addResponseHeader("second-response", "second-response-header"))
                        .uri("http://localhost:8082"))
                .build();
    }
}

```

FilterConfig 클래스는 gatewayRoutes() 메서드를 정의하고 있다. 이 메서드는 RouteLocatorBuilder를 인자로 받아서 RouteLocator를 반환한다. RouteLocatorBuilder는 빌더 패턴을 사용하여 스프링 클라우드 게이트웨이의 라우트를 생성한다.

builder.routes()는 라우트를 정의하는 메서드이다.
이 메서드를 호출하면, path() 메서드를 사용하여 각각의 경로와 URI를 매핑시키는 라우팅 규칙을 정의할 수 있다.

filters() 메서드는 라우트에 적용할 필터를 정의한다.

이렇게 application.yml이 아니라 자바 코드에 의해서 라우팅 정보를 추가할 수도 있다.

```yml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/**
          filters:
            - AddRequestHeader=first-request, first-request-header2
            - AddResponseHeader=first-response, first-response-header2
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
          filters:
            - AddRequestHeader=second-request, second-request-header2
            - AddResponseHeader=second-response, second-response-header2
```

위와 같이 application.yml에서도 필터를 추가할 수 있다.

### Spring Cloud Gateway - Custom Filter 적용

사용자 정의 필터

```java
@Component
@Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {
    public CustomFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // Custom Pre Filter. Suppose we can extract JWT and perform perform Authentication
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServletHttpResponse response = exchange.getResponse();

            log.info("Custom PRE filter: request uri -> {}", request.getId());
            // Custom Post Filter. Suppose we can call error response handler based on error code.
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                log.info("Custom POST filter: response code -> {}", response.getStatusCode());
            }));
        }
    }

    public static class Config {
        // Put the configuration properties
    }
}

```

Custom 필터는 반드시 AbstractGatewayFilterFactory를 상속 받아서 등록하면 된다.

apply 메서드에 작동하고자 하는 내용을 기술.

```yml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/**
          filters:
            #            - AddRequestHeader=first-request, first-request-header2
            #            - AddResponseHeader=first-response, first-response-header2
            - CustomFilter
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
          filters:
            #            - AddRequestHeader=second-request, second-request-header2
            #            - AddResponseHeader=second-response, second-response-header2
            - CustomFilter
```

application.yml 파일에서 CustomFilter를 추가해 주도록 하자

### Spring Cloud Gateway - Global Filter 적용

- Global Filter는 어떠한 라우트 정보가 실행된다고 하더라도 공통적으로 실행될 수 있는 필터
- Global Filter는 호출되는 과정상 모든 필터의 가장 첫번째로 실행이 되고 가장 마지막에 종료가 된다.

Global Filter 생성

```java
@Component
@Slf4j
public class GlobalFilter extends AbstractGatewayFilterFactory<GlobalFilter.Config> {
    public GlobalFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // Custom PreFilter
        return ((exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            log.info("Global Filter baseMessage: {}}", config.getBaseMessage());

            if (config.isPreLogger()) {
                log.info("Global Filter Start: request id -> {}", request.getId());
            }

            // Global Post Filter
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPostLogger()) {
                    log.info("Global Filter End: response code -> {}", response.getStatusCode());
                }

            }));
        });
    }

    @Data
    public static class Config {
        private String baseMessage;
        private boolean preLogger;
        private boolean postLogger;
    }
}
```

- application.yml 코드 추가

```yml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      default-filters:
        - name: GlobalFilter
          args:
            baseMessage: Spring Cloud Gateway Global Filter
            preLogger: true
            postLogger: true
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/**
          filters:
            #            - AddRequestHeader=first-request, first-request-header2
            #            - AddResponseHeader=first-response, first-response-header2
            - CustomFilter
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
          filters:
            #            - AddRequestHeader=second-request, second-request-header2
            #            - AddResponseHeader=second-response, second-response-header2
            - CustomFilter
```

- baseMessage: 필터에서 사용될 메시지를 설정한다.
- preLogger, postLogger: 각각 사전 처리와 후처리 시 로깅을 수행할 지 여부를 설정한다.

### Spring Cloud Gateway - Logging Filter 적용

```java
@Component
@Slf4j
public class LoggingFilter extends AbstractGatewayFilterFactory<LoggingFilter.Config> {
    public LoggingFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
//        // Custom PreFilter
//        return ((exchange, chain) -> {
//            ServerHttpRequest request = exchange.getRequest();
//            ServerHttpResponse response = exchange.getResponse();
//
//            log.info("Global Filter baseMessage: {}}", config.getBaseMessage());
//
//            if (config.isPreLogger()) {
//                log.info("Global Filter Start: request id -> {}", request.getId());
//            }
//
//            // Global Post Filter
//            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
//                if (config.isPostLogger()) {
//                    log.info("Global Filter End: response code -> {}", response.getStatusCode());
//                }
//
//            }));
//        });

        GatewayFilter filter = new OrderedGatewayFilter((exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            log.info("Logging Filter baseMessage: {}}", config.getBaseMessage());

            if (config.isPreLogger()) {
                log.info("Logging PRE Filter: request id -> {}", request.getId());
            }

            // Global Post Filter
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPostLogger()) {
                    log.info("Logging POST Filter: response code -> {}", response.getStatusCode());
                }

            }));
        }, Ordered.HIGHEST_PRECEDENCE);

        return filter;
    }

    @Data
    public static class Config {
        private String baseMessage;
        private boolean preLogger;
        private boolean postLogger;
    }

}
```

OrderedGatewayFilter를 사용하여 Filter Chain에서의 우선순위를 높인다.

```yml
filters:
  - name: CustomFilter
  - name: LoggingFilter
    args:
      baseMessage: Hi, there.
      preLogger: true
      postLogger: true
```

추가적인 파라미터를 넣을라면
name: 옵션을 추가해야함

### Spring Cloud Gateway - Eureka 연동

apigateway-service, first-service, second-service application.yml eureka 등록

```
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka
```

apigateway-service 라우팅 정보 변경

```yml
routes:
  - id: first-service
    uri: lb://MY-FIRST-SERVICE
    predicates:
      - Path=/first-service/**

  - id: second-service
    uri: lb://MY-SECOND-SERVICE
    predicates:
      - Path=/second-service/**
```

기존에는 http, ip, port 번호로 명시를 해줬는데, port 번호를 명시한다는건 그 port번호만 사용한다는 얘기.
우리가 설정을 할 때 랜덤 port로 설정, 랜덤 port로 설정한다는 얘기는 자유롭게 인스턴스를 계속 늘리거나 빼거나 할 수 있는 작업을 의미하는건데 이렇다보니 port번호가 몇번으로 지정될지 모르는 상황.
따라서 api gateway service에서는 EUREKA에 naming으로 등록되어진 이름(MY-FIRST-SERVICE)을 가지고 포워딩을 시켜주겠다는 의미

#### Environment 객체, @Value 어노테이션

- Environment: Spring의 환경 설정 정보를 관리하는 객체.
  이 객체는 Spring의 Application Context가 로딩될 때 생성되어서, Bean으로 등록되어 사용할 수 있다. Environment 객체를 사용하면 환경 변수, 시스템 프로퍼티, Spring 설정 등을 읽어올 수 있다.
- @Value: Spring Framework에서 제공하는 어노테이션 중 하나로 Spring 설정 파일에서 정의한 값을 가져올 수 있다.
  이를 통해, 설정파일에서 값을 설정하고 코드에서 이를 사용할 수 있다.
