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

### User Microservice 인증과 권한 기능 부여

```java
@Data
public class RequestLogin {
    @NotNull(message = "Email cannot be null")
    @Size(min = 2, message = "Email not be less than two characters")
    @Email
    private String email;

    @NotNull(message = "Password cannot be null")
    @Size(min = 8, message = "Password must be equals or grater than 8 characters")
    private String password;
}
```

Request로 전달받을 login 데이터

```java
@Slf4j
public class AuthenticationFilter extends UsernamePasswordAuthenticationFilter {
}
```

UsernamePasswordAuthenticationFilter를 상속받은 AuthenticationFilter.java 생성

```java
@Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
}

@Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
}
```

attemptAuthentication, successfulAuthentication 메서드 구현(오버라이드)

```java
@Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
        try {
            RequestLogin creds = new ObjectMapper().readValue(request.getInputStream(), RequestLogin.class);

            return getAuthenticationManager().authenticate(
                    new UsernamePasswordAuthenticationToken(
                            creds.getEmail(),
                            creds.getPassword(),
                            new ArrayList<>()));

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

- request.getInputStream()에서 InputStream으로 받는이유?
  : 우리가 전달시키고자 하는 로그인의 값은 POST 형태로 전달이 됨 POST로 전달이 되는것은 RequestParameter 정보로 받을 수 없기 때문에 InputStream()으로 처리해주면 수작업으로 데이터가 어떤게 들어왔는지 처리해 볼 수 있다.
- UsernamePasswordAuthenticationToken: 사용자가 입력했던 이메일과 아이디 값을 SpringSecurity에서 사용할 수 있는 값으로 변환하기 위해서 UsernamePasswordAuthenticationToken으로 바꿔줄 필요가 있음
- getAuthenticationManager().authenticate()을 통해 사용자가 입력한 Email과 Password를 인증하는 작업을 함

```java
@Configuration
// final 필드만을 가진 생성자를 만들어주는 Lombok 어노테이션
@RequiredArgsConstructor
// Spring Security를 사용할 것임을 선언
@EnableWebSecurity
public class SecurityConfig {

    private final UserService userService;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    private final Environment env;

    // Spring 컨테이너에 해당 메서드가 생성한 객체를 등록
    // Spring Security 필터 체인의 역할
    // HttpSecurity 객체를 매개변수로 받아 Spring Security 설정을 구성
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        AuthenticationManager authenticationManager = getAuthenticationFilter(http);

        // csrf 보호 기능 비활성화
        http.csrf().disable();
//        http.authorizeRequests().antMatchers("/users/**").permitAll();
        // 요청에 대한 인증 규칙 정의
        http.authorizeRequests()
                // URL 패턴 설정, 해당 패턴에 대한 인증을 통과할 수 있는 권한 부여
                .antMatchers("/actuator/**").permitAll() // actuator permitAll
                .antMatchers("/error/**").permitAll()
                .antMatchers("/**").hasIpAddress("내 IP")
                .and()
                .authenticationManager(authenticationManager)
                .addFilter(getAuthenticationFilter(authenticationManager));

        // frame load 하게 해줌
        http.headers().frameOptions().disable();

        // HttpSecurity 객체를 반환하여 Spring Security가 적용될 수 있도록 함
        return http.build();
    }


    // 인증을 수행하는 AuthenticationManager 객체를 생성하는 메서드
    // 사용자가 전달을 했던 UserName과 Password를 가지고 로그인 처리를 해줌
    private AuthenticationManager getAuthenticationFilter(HttpSecurity http) throws Exception {
      // 인증을 수행하기 위한 여러 구성을 설정할 수 있는 빌더 클래스
        AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
        // 사용자 정보를 가져오는 Service 클래스를 지정
        // 암호화 방식 지정
        authenticationManagerBuilder.userDetailsService(userService).passwordEncoder(bCryptPasswordEncoder);
        return authenticationManagerBuilder.build();
    }

    // 사용자의 인증 정보를 검증하여 인증 토큰을 생성하는 역할을 함.
    // 생성된 인증 토큰은 Spring Security에서 인증을 위해 사용됨
    private AuthenticationFilter getAuthenticationFilter(AuthenticationManager authenticationManager) {
        return new AuthenticationFilter(authenticationManager, userService, env);
    }


}

```

userDetailDetailService를 사용하기 위해서는
UserService 인터페이스에서 UserDetailService를 상속해야함

```java
public interface UserService extends UserDetailsService {
    UserDto createUser(UserDto userDto);

    UserDto getUserByUserId(String userId);
    Iterable<UserEntity> getUserByAll();
}
```

```java
@Service
public class UserServiceImpl implements UserService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserEntity userEntity = userRepository.findByEmail(username);

        if(userEntity == null)
            throw new UsernameNotFoundException(username);

        return new User(userEntity.getEmail(), userEntity.getEncryptedPwd(), true, true, true, true, new ArrayList<>());
    }
```

이 코드는 Spring Security에서 인증을 수행하기 위해 사용자 정보를 가져오는 메서드이다. UserDetailsService 인터페이스를 구현한 클래스에서 사용자 정보를 가져오는 메서드이며, loadUserByUsername() 메서드는 username을 매개변수로 받아서 해당 사용자 정보를 가져온다.

조회한 사용자 정보가 null이 아닌 경우, User 객체를 생성하여 반환한다. 이 객체는 Spring Security에서 사용되는 인증 정보 객체이다. 생성자의 매개변수로 사용자의 이메일, 암호화된 비밀번호, 사용 가능 여부, 계정 만료 여부, 자격 증명(비밀번호) 만료 여부, 계정 잠금 여부, 권한 목록을 전달한다.

new ArrayList()는 사용자의 권한 목록을 나타내며, 현재 이 메서드에서는 권한 목록을 지정하지 않았으므로 빈 리스트를 전달한다.

```yml
routes:
        - id: user-service
            uri: lb://USER-SERVICE
            predicates:
              - Path=/user-service/**
              - Method=POST
            filters:
              - RemoveRequestHeader=Cookie
              - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: user-service
            uri: lb://USER-SERVICE
            predicates:
              - Path=/user-service/users
              - Method=POST
            filters:
              - RemoveRequestHeader=Cookie
              - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: user-service
            uri: lb://USER-SERVICE
            predicates:
              - Path=/user-service/**
              - Method=GET
            filters:
              - RemoveRequestHeader=Cookie
              - RewritePath=/user-service/(?<segment>.*), /$\{segment}
```

- RemoveRequestHeader=Cookie: POST로 전달되는 데이터를 매번 새로운 데이터로 인신하기 위해서 요청 헤더 중 Cookie 헤더 삭제
- RewritePath=/user-service/(?<segment>.\*), /$\{segment}: 사용자가 /user-service를 알 필요 없이 API 엔드포인트에 직접 액세스할 수 있음

```java
@RestController
@RequestMapping("/")
public class UserController {
}
```

위의 라우팅 설정을 통해 사용자 측에서 userservice를 생략할 수 있다.

#### User 로그인 처리 과정

1. 사용자 로그인 (email, password)
2. AuthenticationFilter에 전달

- attemptAuthentication() 처리
- 입력되어진 email과 password를 UsernamePasswordAuthenticationToken 형태로 바꿔서 사용한다.

3. UserDetailService를 구현하고 있는 클래스에 loadUserByUsername()이 실행 UserRepository를 통해 UserEntity의 형태로 가져와 User라는 형태로 변환해서 가져온다.
4. 모든 과정이 끝나서 정상적으로 로그인이 되어진걸로 확인이 된다고 하면 successfulAuthentication()에서 해당하는 이 값을 가지고 토큰을 발행을 해야되는데 문제는 successAuthentication 안에서 사용자의 정보를 확인하기 위해서는 authResult안에 포함되어 있는 객체의 Username을 가지고 와야만 처음에 입력한 이메일을 가지고올 수 있다.
   우리가 하고자하는 작업은 해당하는 이메일 정보를 토대로해서 토큰을 생성할건데 이메일가지고 토큰을 만드는 것이 아니라 userId를 가지고 토큰을 만들 것이다.
   현재 반환값은 UserEmail 값을 가지고 있는 상태이기 때문에 UserEmail 값을 통해 userId값을 바로 가져오는 것은 안되기 때문에
   UserEmail로 다시한번 데이터베이스에서 실제 Object 값을 가지고 올 것이다.
   그 다움 이렇게 반환되어진 객체(UserDTO)안에 포함되어 있는 userID를 통해 JWT를 생성할 것이다.
   이렇게 생성된 토큰값을 response의 헤더값에 저장을 시킬 것이다.
   그럼 헤더 값이 원래 있던 클라이언트한테 반환이 된다.

#### JWT 생성

```yml
token:
  # 만료시간(하루)
  expiration_time: 86400000
  # 특수한 토큰을 만들 때 사용되어짐
  secret: user_token
```
