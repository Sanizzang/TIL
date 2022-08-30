### Gradle은 무엇을 하는가?

Gradle은 CI/CD를 위한 아래 Task들을 자동화 시켜주는 Build Tool이다.

- Compile
  : Kotlin 파일이나 Java파일을 바이트 코드로 변환해주는 작업

- Test
  : 어플리케이션이 제대로 동작할지에 대한 Test를(유닛 테스트, UI 테스트 등) 지원한다.

- Packaging
  : 코드를 패키징해 jar 파일이나 war 파일로 만들어주는 것을 뜻함.

- Deploy & Run
  : 만들어진 스프링을 돌려 서버를 실행해주는 것

### Gradle의 강점

#### 직관적인 코드와 자동완성

Gradle은 Maven이나 Ant 등의 다른 빌드 툴들에 비해 매우 직관적이다. 특히 Maven의 경우 모든 파일들이 xml로 이루어져 있어서
사실상 매우 알아보기가 힘들고, 문서를 찾기도 힘들다. Gradle은 Markup Language가 아닌 Groovy나 Kotlin으로 짤 수 있어서
매우 가독성이 뛰어나다.

몇년 전까지는 Groovy만 지원해서 자동완성과 코드 찾기가 안되어 매우 불폈했는데,
최근에는 Kotlin DSL을 이용해 사용할 수 있게 되어서 가독성 또한 매우 올라갔고 커스텀 스크립트를 정말 손쉽게 짤 수 있게 되었다.

#### 다양한 Repository 사용 가능

Gradle은 다양한 Repository의 사용이 가능하다. Maven Repository(Google Maven Repository, Maven Central 등),
JCenter(안드로이드에서 최근에 Deprecated 되었다) 외 커스텀 파일 저장 공간 등 다양한 저장 공간을 사용하기 위해
단순히 Repository에 대한 정의만 해주면 된다.

#### 각 작업에 필요한 라이브러리들만을 가져오는 작업

프로젝트를 만든 후 build.gradle을 보면 아래와 같은 여러 라이브러리에 대한 implementation이 있는 것을
확인할 수 있다. 여기서 implementation은 라이브러리를 가져오는 것을 뜻하는데,
implementation이 들어가는 자리에 다양한 implementation이 있는 것을 확인할 수 있다.
즉, 원하는 Scope 별로 다른 라이브러리를 손쉽게 가져올 수 있도록 한다.

### Gradle이 빌드 속도가 빠른 이유

#### 점진적 빌드(Incremental Build)

Gradle의 점진적 빌드는 이미 빌드된 파일들을 모두 다시 빌드하는 것이 아닌 바뀐 파일들만 빌드하는 것을 뜻한다.

#### Build Cache

만약 두 개 이상의 빌드가 돌아가고, 하나의 빌드에서 사용되는 파일들이 다른 빌드들에 사용된다면
Gradle은 빌드 캐시를 이용해 이전 빌드의 결과물을 다른 빌드에서 사용할 수 있다.
이로 인해 다시 빌드하지 않아도 되므로 빌드 시간이 줄어들게 된다.

#### Daemon Process

또한 Gradle은 데몬 프로세스를 지원한다. 데몬 프로세스는 서비스의 요청에 응답하기 위해 오래 동안 살아있는 프로세스인데
Gradle의 데몬 프로세스는 메모리 상에 결과물을 보관한다. 이로 인해 한 번 빌드된 프로젝트는 다음 빌드에서 매우 적은
시간만 소요된다.
