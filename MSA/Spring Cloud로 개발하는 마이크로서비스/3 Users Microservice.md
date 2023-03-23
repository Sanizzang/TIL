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
