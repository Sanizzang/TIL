### H2 데이터베이스 연동

```groovy
// https://mvnrepository.com/artifact/com.h2database/h2
runtimeOnly group: 'com.h2database', name: 'h2', version: '2.1.214'
```

build.gradle에 h2 dependency 추가

```yml
spring:
  application:
    name: user-service
  h2:
    console:
      enabled: true
      # 외부에서 접근 가능
      settings:
        web-allow-others: true
      # 접속하고자하는 웹 브라우저에서 h2 콘솔의 주소
      path: /h2-console
```

application.yml h2 설정 추가

### Spring Security 연동

#### Spring Security

- Spring Security는 Authentication(인증), Authorization(권한) 기능을 다 제공해줌.

```
Step1: 애플리케이션에 spring security jar을 Dependency에 추가
Step2: WebSecurityConfigurerAdapter를 상속받는 Security Configuration 클래스 생성
Step3: Security Configuration 클래스에 @EnableWebSecurity 추가
Step4: Authentication -> configure(Authentication ManagerBuilder auth) 메서드를 재정의
Step5: Password encode를 위한 BCryptPasswordEncoder 빈 정의
Step6: Authorization -> configure(HttpSecurity http) 메서드를 재정의
```

#### Step1: 애플리케이션에 spring security jar을 Dependency에 추가

```groovy
dependencies {
	implementation group: 'org.springframework.boot', name: 'spring-boot-starter-security'
}
```

#### Step2: WebSecurityConfigurerAdapter를 상속받는 Security Configuration 클래스 생성

#### Step3: Security Configuration 클래스에 @EnableWebSecurity 추가

#### Step4: Authentication -> configure(Authentication ManagerBuilder auth) 메서드를 재정의

Spring Boot 최신 3.XX 버전 Security 설정이 변경됨에 따라
Spring Security Configuration 설정 내용이 변경되었다.
WebSecurityConfigurerAdapter 클래스가 deprecated되었는데,
해당 클래스를 상속 받아 config 메소드를 구현하는 대신
SecurityFilterChain을 반환하고 직접 Bean으로 등록하도록 설정 방법이 바뀌었다.

```java
// 이 클래스가 스프링 설정 클래스임을 나타냄
@Configuration
// 이 클래스에서 웹 보안을 사용하도록 설정
@EnableWebSecurity
public class WebSecurity {
    // HttpSecurity 객체를 사용하여 웹 애플리케이션의 보안 설정을 구성
    @Bean
    protected SecurityFilterChain config(HttpSecurity http) throws Exception {
        // CSRF(Cross-Site Request Forgery) 방어 기능을 비활성화한다.
        http.csrf().disable();
        // X-Frame-Options 헤더를 비활성화 한다.
        // X-Frame-Options 헤더는 웹 페이지가 다른 페이지의 <iframe> 또는 <frame> 내에서 랜더링되는 것을 제어하는 보안 기능이다.
        http.headers().frameOptions().disable();
        // HTTP 요청에 대한 인증 및 권한을 설정.
        http.authorizeHttpRequests(authorize -> authorize
                // "/users/**" URL 패턴에 대해 모든 사용자가 접근할 수 있도록 허용. 이는 인증이 필요하지 않은 엔드포인트에 대한 접근을 허용하고자 할 때 사용된다.
                .requestMatchers("/users/**").permitAll()
                // H2 데이터베이스 콘솔에 대한 요청을 모든 사용자가 접근할 수 있도록 허용한다.
                // 이 설정은 일반적으로 개발환경에서만 사용되며, 실제 프로덕션 환경에서는 보안상 사용되지 않는다.
                .requestMatchers(PathRequest.toH2Console()).permitAll()
        return http.build();
    }
}
```

요약하자면, 이 코드는 Spring Security를 사용하여 웹 애플리케이션의 보안을 설정하고, 특정 URL 패턴에 대해
인증을 생략하도록 구성한다. 또한 CSRF 방어 기능과 X-Frame-Options 헤더를 비활성화하여,
웹 애플리케이션의 보안 요구 사항에 맞게 설정을 변경한다.

#### Step5: Password encode를 위한 BCryptPasswordEncoder 빈 정의

```java
// 이 클래스가 Spring Boot 애플리케이션의 시작지점임을 나타냄
@SpringBootApplication
// 이 클래스에서 서비스 디스커버리 클라이언트를 활성화한다.
// 이를 통해 마이크로서비스 아키텍처에서 이 애플리케이션 인스턴스를 서비스 디스커버리 서버(Neflix Eureka)에 등록하고
// 다른 마이크로서비스와 통신할 수 있게 된다.
@EnableDiscoveryClient
public class UserServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(UserServiceApplication.class, args);
	}

  // BCryptPasswordEncoder 객체를 생성하고, 스프링 컨테이너에서 관리할 수 있도록 빈으로 등록
  // 이 객체는 비밀번호 암호화에 사용된다.
	@Bean
	public BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
}
```

```java
// 스프링 서비스로 등록
@Service
public class UserServiceImpl implements UserService {
    // 사용자 정보를 저장 및 조회하기 위한 userRepository
    UserRepository userRepository;
    // 비밀번호 암호를 위한 BCryptPasswordEncoder
    BCryptPasswordEncoder passwordEncoder;

    // 의존성 주입
    @Autowired
    public UserServiceImpl(UserRepository userRepository, BCryptPasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    // 사용자를 생성하는 기능
    @Override
    public UserDto createUser(UserDto userDto) {
        // UUID를 사용하여 고유한 ID 생성
        userDto.setUserId(UUID.randomUUID().toString());

        // ModleMapper 객체 생성 및 매핑 전략을 STRICT로 설정
        // 소스와 대상 객체의 필드 이름이 정확히 일치해야만 매핑이 이루어짐
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);
        // UserDto -> UserEntity로 변환
        UserEntity userEntity = mapper.map(userDto, UserEntity.class);
        // 비밀번호를 암호화하여 UserEntity의 encryptedPwd 필드에 저장
        userEntity.setEncryptedPwd(passwordEncoder.encode(userDto.getPwd()));
        // UserRepository를 사용하여 데이터베이스에 저장
        userRepository.save(userEntity);

        // UserEntity를 다시 modelMapper를 사용하여 UserDto로 변환
        UserDto returnUserDto = mapper.map(userEntity, UserDto.class);

        return returnUserDto;
    }
}
```
