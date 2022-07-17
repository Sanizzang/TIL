### 1. 자바 프로그램 실행 과정

(https://velog.velcdn.com/images/sanizzang00/post/e5696ea3-c3e4-4a05-a209-e962b21e1b85/image.png)

> Java언어로 프로그래밍된 파일을 Java컴파일러가 가상 기계어 파일인 Java클래스 파일로 만든다. 다시 말해, 소스 코드를 Java바이트 코드로 번역한다. 이후 Java바이트 코드를 JVM이 읽고 실행하게 된다.

#### 자바 바이트 코드

- JVM이 이해할 수 있는 언어로 변환된 자바 소스 코드를 의미한다. 자바 컴파일러에 의해 변환되는 코드의 명령어 크기가 1바이트라서 자바 바이트 코드라고 불린다.

- 이러한 자바 바이트 코드의 확장자는 .class이며 자바 바이트 코드는 자바 가상 머신만 설치되어 있으면, 어떤 운영체제에서 라도 실행될 수 있다.

### JVM이란?

(https://velog.velcdn.com/images/sanizzang00/post/fdaa0dd0-6161-4040-8766-6a6edc1dfb2e/image.png)

- JVM은 JavaVirtual Machine의 줄임말로 write once, run everywhere 즉 OS마다 따로 코드를 작성해야 하는 번거로움 없이 Java가 플랫폼에 독립적일 수 있게 만들어준다.

- 예를 들어 C 프로그램은 바로 기계어로 컴파일하므로 HW 기종에 맞게 컴파일되어야한다. 이를 '플랫폼에 종속적'이다 라고 한다. 반면 Java프로그램은 중간 단계 언어로 컴파일하여 JVM만 각 OS에 설치되어 있다면 HW 기종에 상관없이 단 한 번만 컴파일 하면 된다.

- 간단히 말해 JVM은 Java 클래스 파일을 로드하고 바이트 코드를 해석하며, 메모리 등의 자원을 할당하고 관리하며 정보를 처리하는 작업을 하는 프로그램이다.

(https://velog.velcdn.com/images/sanizzang00/post/45529cd2-8912-4099-b854-ab4b09e28d66/image.png)

#### Java Compiler

Java Source파일을 JVM이 해석할 수 있는 Java Byte Code(.class)로 변경한다. 일반적인 윈도우 프로그램의 경우, Compile 이후 Assembly 언어로 구성된 파일이 생성된다.

#### Class Loader

JVM내로 .class파일들을 Load한다. Loading 된 클래스들을 Runtime Data Area에 배치한다. 일반적인 윈도우 프로그램의 경우 Load 과정은 OS가 주도한다.

#### Execution Engine

Loading된 클래스의 Bytecode를 해석한다. 이 과정에서 ByteCode가 BinaryCode로 변경된다. 일반적인 윈도우 프로그램의 경우 Assembier가 Assembly 언어로 쓰인 코드파일을 BinaryCode로 변경한다.

#### Runtime Data Area

JVM이 프로세스로써 수행되기 위해 OS로부터 할당받는 메모리 영역이다. 저장 목적에 따라 다음과 같이 5개로 나눌 수 있다.

- Method Area
  모든 Thread에게 공유된다. 클래스 정보, 변수 정보, Method정보, static변수 정보, 상수 정보 등이 저장되는 영역

- Heap Area
  모든 Thread에게 공유된다. new 명령어로 생성된 인스턴스와 객체가 저장되는 구역, 공간이 부족해지면 Garbage Collection이 실행된다.

- Stack Area
  각 스레드마다 하나씩 생성된다. Method안에서 사용되는 값들(매개변수, 지역변수, 리턴 값 등)이 저장되는 구역, 메서드가 호출될 때 LIFO로 하나씩 생성되고, 메서드 실행이 완료되면 LIFO로 하나씩 지워진다.

- PC Register
  각 스레드마다 하나씩 생성된다. CPU의 Register와 역할이 비슷하다. 현재 수행중인 JVM 명령의 주소값이 저장된다.

- Native Method Stack
  각 스레드마다 하나씩 생성된다. 다른 언어(C/C++ 등)의 메서드 호출을 위해 할당되는 구역 언어에 맞게 Stack이 형성되는 구역이다. JNI(Java Native Interface)라는 표준 규약을 제공한다.
