하나의 어플리케이션 내에서 모든 것을 개발하고 서로 메소드
호출로서 이루어지는 모놀리식과 방식과 비교하여 마이크로서비스 방식은 물리적으로 분산된 시스템, 서비스 간의 통신이 필수적이다.
이러한 마이크로서비스간의 통신이 어떠한 종류가 있는지 살펴보고 대표적인 Http 통신 방법인 RestTemplate의 사용법에 대해서 알아본다.

Spring Cloud로 개발된 어플리케이션, 마이크로서비스들 간에는 동기방식과 비동기방식으로 통신을 할 수 있다. 동기 방식이라는 것은 기존의 웹 서비스를 만들었을 때 하나의 클라이언트 요청사항이 들어오면 해당하는 요청이 다 끝날 떄까지 다른 작업을 할 수 없어 다른 클라이언트의 요청을 처리할 수 없었다.

AMQP라는 프로토콜을 이용을해서 각 마이크로서비스간에 비동식으로 통신하는 것에대해 알아봤었다.
우리는 Spring Configuration의 정보 값을 각각의 마이크로 서비스가 순차적으로 데이터를 동기화하는것이 아니라 일단 연결되어 있는 모든 마이크로 서비스들 한테 변경된 사항을 전달해 줄 수 있는 사용하기 위해서 비동기 방식의 AMQP라는 프로토콜을 사용해 봤다.
앞의 프로젝트를 예시로 들자면 Eureka 서비스에 orderService가 2개가 등록되어 있다고 해보자 주문이 늘어서 주문을 처리하는 마이크로서비스가 2개 이상 기동되어있다고 가정. 이제 클라이언트로부터 요청을 받아서 UserService가 작동이 된다. UserService는 주문서비스로부터 데이터를 가져와야되는 상황이다. UserService는 Eureka에게 orderService의 존재를 물어본다. 어떤 서비스를 사용하면 되는지, 어디에 서비스가 존재하는지, UserService는 Eureka로 부터 OrderService 정보를 받아서 직접적으로 해당하는 OrderService를 호출하는 방식이었다.

우리가 처음에 Spring Boot로 만들어져있는 Application에서 웹 Application을 구성하고자 할 때, 서버 포트를 지정할 수 있었다. 서버 포트에 랜덤포트를 사용하기 위해서 0이라는 값을 집어넣어봤었는데, 랜덤포트를 이용하면 포트가 랜덤하게 기동이 되고 Eureka 서비스에서 그런 랜덤 포트로 기동되어있는 동일한 마이크로서비스를 호출할 때 가장 기본적인 방식으로 라운드로빈이라고 해서 순차적으로 데이터를 호출했다.
아니면 지역, 시간 등을 통해 order service를 분산시킬 수 있었음. 어쨌거나 Eureka 서비스를 통해 order service의 존재를 물어보고 마이크로서비스는 그런 order service의 정보를 얻어서 호출하는 방식이 있었다.
2번째 마이크로서비스 간에 데이터를 호출할 수 있는 방식 중에서 전통적으로 많이 사용되었던 RestTemplate이라는 API를 사용할 수도 있다.
RestTemplate이라는 것은 마이크로서비스라던가 Spring Cloud 기술에 도입된 것이 아니라 기존에 자바로 만들어진 웹 어플리케이션 간에서 HTTP 프로토콜을 이용해서 GET 방식이라던가, POST 방식이라던가 이러한 방식으로써 또 다른 서비스, 또 다른 API를 호출하기 위해서 사용되었던 방식이다.

RestTemplate이라는 인스턴스를 생성하고 해당하는 인스턴스에다가 필요로하는 GET이나 POST 메소드를 정의하고 파라미터가 필요하다고 하면 파라미터 값도 받아온다.

UserService는 RestTemplate이라는 객체를 이용해서 서비스를 대신 처리해줄 수 있는 대리자 역할을하게 할 것이다.
클라이언트가 사용자의 정보역할을 요청 -> user-service는 REST TEMPLATE을 이용을 해서 Eureka 서비스에서 얻어왔던 order service 정보를 가지고 직접 호출을하게 될 것이다.

#### RestTemplate 사용

- RestTemplate 빈 추가

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(UserServiceApplication.class, args);
	}

	@Bean
	public BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}

	@Bean
	AuthenticationManager authenticationManager(AuthenticationConfiguration authConfiguration) throws Exception {
		return authConfiguration.getAuthenticationManager();
	}

	@Bean
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}
}

```

- Config-service
  - user-service.yml

```yml
order_service:
  url: http://127.0.0.1:8000/order-service/%s/orders
```

- UserController
  - getUserByUserId: 사용자 ID를 기반으로 사용자 정보를 검색하고, 해당 사용자의 주문 목록을 가져와서 'UserDto' 객체에 추가한 후 반환

```java
    @Override
    public UserDto getUserByUserId(String userId) {
        // 사용자 정보를 나타내는 JPA 엔티티
        // JPA repository를 사용하여 사용자 ID를 기반으로 사용자 정보를 가져옴
        UserEntity userEntity = userRepository.findByUserId(userId);

        if (userEntity == null)
            throw new UsernameNotFoundException("User not found");

        // userEntity 객체를 UserDto 객체로 매핑하여 반환
        UserDto userDto = new ModelMapper().map(userEntity, UserDto.class);

//        List<ResponseOrder> orders = new ArrayList<>();

        /* Using as RestTemplate */
        // 주문 서비스의 엔드포인트 URL을 동적으로 생성
        // URL의 일부로 사용되는 userId는 메소드의 매개변수로 전달
        // 환경 변수에서 order_service.url 키에 해당하는 값을 가져와 URL 템플릿의 %s 위체에 대체
        String orderUrl = String.format(env.getProperty("order_service.url"), userId);
        ResponseEntity<List<ResponseOrder>> orderListResponse =
                // restTemplate.exchange RestTemplate 클래스를 사용하여 HTTP 요청을 보내고, 응답을 받아옴
                // 이 코드에서는 주문 서비스로 요청을 보내서 해당 사용자의 주문 목록을 가져온다.
                // ParameterizedTypeReference<List<Response>>()는 RestTemplate에서 제공하는 제네릭 타입을 사용하는 방법으로 List<ReponseOrder> 형식의 응답을 받기 위해 사용된다.
                restTemplate.exchange(orderUrl, HttpMethod.GET, null, new ParameterizedTypeReference<List<ResponseOrder>>() {
        });

        List<ResponseOrder> orderList = orderListResponse.getBody();

        userDto.setOrders(orderList);

        return userDto;
    }
```

#### order service url를 microservice name으로 바꾸기

- RestTemplate @LoadBalanced 추가

```yml
	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}
```

- Config server
  - user-service.yml 변경

```yml
order_service:
  url: http://ORDER-SERVICE/order-service/%s/orders
```

### 마이크로서비스 호출 2 - FeignClient

- FeignClient -> Http Client
  - REST Call을 추상화 한 Spring Cloud Netflix 라이브러리
- 사용방법
  - 호출하려는 HTTP Endpoint에 대한 Interface를 생성
  - @FeignClient 선언
- Load balanced 지원

#### dependency 추가

```groovy
	// https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-openfeign
	implementation group: 'org.springframework.cloud', name: 'spring-cloud-starter-openfeign'
```

#### UserServiceApplication 어노테이션 추가

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class UserServiceApplication {
}
```

- @EnableFeignClients 어노테이션 추가

#### OrderService Client 생성

```java
@FeignClient(name="order-service")
public interface OrderServiceClient {

    @GetMapping("/order-service/{userId}/orders")
    List<ResponseOrder> getOrders(@PathVariable String userId);

}
```

#### UserSeriviceImpl 생성자 주입 및 데이터 요청

```java
    OrderServiceClient orderServiceClient;

    @Autowired
    public UserServiceImpl(UserRepository userRepository,
                           BCryptPasswordEncoder passwordEncoder,
                           Environment env,
                           RestTemplate restTemplate,
                           OrderServiceClient orderServiceClient) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.env = env;
        this.restTemplate = restTemplate;
        this.orderServiceClient = orderServiceClient;
    }

    /* Using feign client */
        List<ResponseOrder> orderList = orderServiceClient.getOrders(userId);

        userDto.setOrders(orderList);
```

#### RestTemplate과 FeignClient 차이점

코드의 양으로 보거나 직관적인 측면에서는 FeignClient를 사용하는 것이 유리하다.
FeignClient는 orderService를 만든 사람이 아닌 경우에는 제대로 파악이 안되는 경우가 있을 수도 있다. 인터페이스 안에 또 다른 어플리케이션에 있는 메소드를 호출하는 것이기 때문에 userService만 놓고 봤을 때는 어떠한 작업을 하는지 명확하게 모를 수 있다.

반면에 RestTemplate 방법은 기존에 있었던 HTTP의 IP와 PORT번호, URI, Endpoint, Http Method, Parameter, 이런 값들을 가지고 전달을 하기 떄문에 자바 클래스에서 또 다른 네트워크를 통해서 또 다른 서비스를 호출하는 방법이다.

### Feign Client에서 예외, 로그 사용

#### FeignClient 사용 시 발생한 로그 추적

- application.yml

```yml
logging:
  level: com.example.userservice.client:DEBUG
```

- 해당하는 패키지에 어떠한 log 레벨로 log를 출력할 것인지 설정

```java

@Bean
public Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;
}
```

- application 메인 클래스에 Logger.Level이라는 반환 값을 하나 가지고 있는 Bean을 설정

- 이 설정만으로 feignClient가 호출이 되었을 때 로그가 출력이 된다.

### ErrorDecoder를 이용한 예외 처리

#### error 패키지 생성,
