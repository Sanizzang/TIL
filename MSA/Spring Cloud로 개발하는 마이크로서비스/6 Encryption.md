### Spring Cloud Config Server의 암호화 복호화 지원

Spring Cloud Config Server는 Jasypt와 같은 암호화 모듈을 지원한다.

Spring Cloud Config Server에서 구성 정보를 암호화하려면, 구성 정보를 암호화할 수 있는 암호화 키를 지정해야 한다.

## 대칭키를 이용한 암호화

### Config Server에 dependency 추가

```groovy
implementation group: 'org.springframework.cloud', name: 'spring-cloud-starter-bootstrap'
```

- bootstrap.yml 생성

```yml
encrypt:
  key: abcdefghijklmnopqrstuvwxyz0123456789
```

암호화를 하기 위해서 key 값이 하나 필요, 이 키를 이용해서 plain text를 다른형태로 바꾸는 작업을 한다.

변경되어진 encrypt를 decrypt하는 작업에서도 key값이 사용되어진다.

대칭키같은 경우 encrypt, decrypt 모두 key값이 같다.

bootstrap.yml 설정정보 값은 자동으로 로딩되어지지 않는다.

api-gateway, user-service에서 등록했던 것처럼 spring-cloud-starter-bootstrap dependency를 추가해야지만 된다.

- POST - 127.0.0.1:8888/encrypt

#### User Microservice의 application.yml, bootstrap.yml 수정 -> Config Server의 user-service.yml로 이동

user microservice의 데이터베이스 비밀번호와 같은 민감한 데이터를 포함하고 있으면 별도의 파일로 분리해서 만든다 -> config server의 user-service.yml

- user-service.yml

```yml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    # 암호화된 값이라는 것을 명시해야함
    password: "{cipher} 암호화 값"

token:
  expiration_time: 864000000
  secret: user_token_native_user_service

gateway:
  ip: 192.168.0.11
```

## 비대칭키를 이용한 암호화

암호화를 할 때 사용하는 키와 복호화를 할 때 사용하는 키를 달리 사용한다.

keytool을 이용해서 원하는 내용의 key를 사용할 수 있는데,
사용하는 알고리즘(RSA), self 인증이라고 해서 키를 본인이 인증합니다. 라는 식의 부가적인 정보를 추가해 넣어야한다.
key를 생성할떄 password는 임의로 생성,
key가 생성이 되고자 하는 keystore 데이터타입

- Public, Private Key 생성 -> JDK keytool 이용
- $ mkdir ${user.home}/디렉토리 명
- keytool -genkeypair -alias apiEncryptionKey -keyalg RSA \
  -dname "CN=Kenneth Lee, OU=API Development, O=joneconsulting.co.kr, L=Seoul, C=KR" \
  -keypass "1q2w3e4r" -keystore apiEncryptionKey.jks -storepass "1q2w3e4r"

위의 명령어는 Java의 keytool을 사용하여 RSA 알고리즘을 사용하는 키 쌍을 생성하고, 그것을 포함하는 keystore 파일을 만드는 명령어이다. 이 명령어를 사용하여 'apiEncryptionKey'라는 별칭(alias)을 갖는 키 쌍을 생성하고, 그것을 'apiEncryptionKey.jks'라는 이름의 keystore 파일에 저장한다.

-alias 옵션은 생성할 키 쌍의 이름을 지정한다. (apiEncryptionKey)

-keyalg 옵션은 사용할 대칭키 알고리즘을 지정한다. (RSA)

-dname 옵션은 인증서를 생성할 때 사용할 Distinguished Name(DN)을 지정한다. DN은 SSL 인증서에서 인증서 주체를 식별하기 위한 정보를 나타내며, 일반적으로 국가, 지역, 조직명, 부서명, 이름 등의 정보를 포함한다.

-keypass 옵션은 생성된 개인키를 보호하기 위한 암호를 지정한다. (1q2w3e4r)

-keystore 옵션은 생성된 키 쌍과 인증서를 저장할 keystore 파일의 경로와 이름을 지정한다.

-storepass 옵션은 keystore 파일을 보호하기 위한 암호를 지정합니다. (1q2w3e4r)

#### 위의 생성된 key로부터 공개키(publicKey)를 끄집어 올 수 있다.

```cmd
keytool -export -alias apiEncryptionKey -keystore apiEncryptionKey.jks -rfc -file trustServer.cer
```

위 명령어는 Java Keytool을 사용해서 apiEncryptionKey.jks라는 Java KeyStore 파일에서 apiEncryptionKey라는 별칭으로 지정된 개인 키를 가져와서 인증서 파일인 trustServer.cer로 내보낸다.

일반적으로 이 명령어는 SSL/TLS 연결을 설정할 때 사용됨며, 클라이언트가 서버의 공개 키를 신뢰할 수 있도록 한다. 이를 위해, 서버는 이 명령어를 사용하여 공개 키의 인증서를 생성하고, 클라이언트는 이 인증서를 신뢰할 수 있는 루트 CA 인증서로 인증한 후, SSL/TLS 연결을 설정한다.

-rfc 옵션은 인증서를 RFC(Request for Comments) 형식으로 내보내도록 지정하는 옵션이다. 이렇게 하면 인증서가 텍스트 형식으로 인코딩되어 인증서를 쉽게읽을 수 있다.

#### 인증서 파일을 다시 jks 파일로 돌려서 사용할수도 있다.

```cmd
keytool -import -alias trustServer -file trustServer.cer -keystore publicKey.jks
```

위 명령어는 publicKey.jks라는 Java KeyStore 파일에 trustServer.cer 파일에서 가져온 인증서를 trustServer라는 별칭으로 추가하는데 사용된다.

#### config-service bootstrap 설정 변경

```yml
encrypt:
  #  key: abcdefghijklmnopqrstuvwxyz0123456789
  key-store:
    locatioon: file:///${user.home}\\D:\\Study\\keystore\\apiEncryptionKey.jks
    password: 1q2w3e4r
    alias: apiEncryptionKey
```
