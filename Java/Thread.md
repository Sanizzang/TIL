## Thread?

: 우리가 컴퓨터 프로그램을 실행시키면 프로그램에 의해 메모리가 할당되게 된다.
이렇게 메모리가 할당이 되고 실행이 된 프로그램을 일컬어 “프로세스”라고 한다.
이 프로세스는 일련의 “루틴”이라는 프로그램에서 반복되는 실행코드를 가지게 된다.
보통은 이 루틴이 1개이기 때문에 일정한 1개의 흐름대로 프로그램이 진행이 된다.
하지만 한 번에 2개 이상의 흐름으로 프로그램을 진행하고 싶을 때 사용하는 것이 “쓰레드”이다.

쓰레드: 프로세스에서 프로그램의 흐름을 형성하는 것

### 쓰레드 생성 방법

1. Thread 클래스 상속
   ![](https://velog.velcdn.com/images/sanizzang00/post/50c91944-22a0-430b-a09f-9b698826dbfc/image.png)

Thread 클래스를 상속하면 당연하게 “run 메소드”를 오버라이드 해야한다.

- 메소드를 run이 아닌 start를 사용하는 이유?
  : 바로 쓰레드를 생성하기 위함
  직접 run을 호출하게 되면 단순하게 메소드만 실행이 되지만,
  start를 호출하게 되면 쓰레드를 생성하여 run을 실행하게 해준다.

위의 코드의 출력결과는 항상 다르다.
-> 쓰레드는 기본적으로 지 멋대로 작동한다.

2. Runnable 인터페이스 구현

![](https://velog.velcdn.com/images/sanizzang00/post/adb28a76-46a4-4813-8b46-a3a37666d127/image.png)

: sleep 메소드를 사용할 때 Thread.sleep으로 사용
-> Thread를 extends 하지 않았기 때문

Thread 쓰레드이름 = new Thread(연동시킬\_객체);
-> Runnable을 implements된 클래스를 쓰레드로 연동시켜주는 것

- Runnable을 따로 만들어둔 이유?
  : “다중 상속”을 위함
  만약 extends로 Thread를 상속하게 되면 다른 클래스를 상속할 수 없게 된다.
  그런데 implements로 Runnable을 구현하게 되면 다른 클래스 하나를 상속할 수 있게 된다.

### 쓰레드 우선순위

: 쓰레드에는 우선순위라는 것이 존재한다.
그 우선순위를 기준으로 쓰레드를 실행하게 된다.
(1이 최저, 10이 최대)

- 우선순위 관련 메소드
  우선순위 설정 메소드: setPriority(순위);
  우선순위 확인 메소드: getPriority();

![](https://velog.velcdn.com/images/sanizzang00/post/72a1c2fd-a05d-4bea-8e65-71e930f6af76/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/0f1209d2-4bb7-4251-a447-906d02998a1d/image.png)

- 자바에서의 쓰레드 우선순위
  : 자바에서 우선순위는 사실상 운영체제에 따라서 달라지게 된다.
  A라는 운영체제에서 최대 우선순위가 5라면 자바에서의 우선순위의 10이 5가된다.
  B라는 운영체제에서 최대 우선순위가 14라면 자바에서 우선순위 10이 14가 된다.
  그래서 해당 운영체제의 우선순위에 대해 알아야 하고,
  상수 “MAX_PRIORITY, NORM_PRIORITY, MIN_PRIORITY”를 사용하여 하는 것이 좋다.

- 우선순위가 같다면?
  -> 먼저 끝난 쓰레드부터 실행된다.

![](https://velog.velcdn.com/images/sanizzang00/post/2de1120a-7dfb-4e11-9d81-aaaf9b321349/image.png)

: 위의 코드를 실행하면 0을 출력함을 볼 수 있다.
-> 쓰레드는 기본적으로 지 멋대로 작동하기 때문에 CalcThread가 run메소드를 실행하기 이전에 main메소드 쓰레드의 System.out.println()이 실행이 되는 것이다.
-> “join” 함수로 해결

![](https://velog.velcdn.com/images/sanizzang00/post/48aee902-c8ea-4c95-a711-80d8431c2366/image.png)

### 쓰레드 라이프 사이클(쓰레드의 생명 주기)

![](https://velog.velcdn.com/images/sanizzang00/post/84b43f04-9b47-48dd-ac73-5ff64689ca45/image.png)
New: 쓰레드가 처음 생성(new)된 상태를 가리켜 “New”라고 한다.
Runnable: 쓰레드가 생성되어 start된 상태로 실행될 준비가 된 상태를 “Runnable”이라고 한다.
Running: Runnable 상태에서 run된 상태
Waiting: Running 도중에 잠깐 중단(종료X)된 상태 (Blocked 또는 Non-Runnable이라고도 한다.)
Dead: run이 완료되어 끝난 상태

### 쓰레드의 동기화(Synchronized)

: 쓰레드의 동기화는 주로 쓰레드가 2개 이상일 때 사용

쓰레드 끼리 값 공유 가능 -> 힙영역(Heap)을 이용

자바에서는 JVM이라고 하는 가상머신 위에서 프로그램이 돌아가게 된다.
이 JVM이 프로그램을 실행하기 위해서 메모리를 구성하게 되는데 다음과 같이 구성한다.
![](https://velog.velcdn.com/images/sanizzang00/post/dcd3ecb1-8d51-4fe6-a295-d9a531f2e741/image.png)
스택(Stack): 메소드에서 사용하는 각종 값들, 리턴 값들을 저장
힙(Heap): 실행되는 클래스에서만 실행가능하고, new로 인스턴스화 된 객체들이 저장
(GC를 사용하여 메모리 반환)
클래스(혹은 메소드): 클래스에서 사용되는 내용들이 저장(메소드, 자잘한 변수들의 이름 등등)
Native Method Stack: 자바 외의 다른 언어로 구현된 메소드를 위한 스택 공간
Register: JVM에서 생성한 명령들의 주소를 저장

: 고로 우리가 어떤 클래스를 new 하게 되면 그 클래스는 힙공간에 상주하게 된다.

- 동기화 처리
  : 동기화 처리된 구간은 다수의 쓰레드에서 처리한다고 할 지라도 한번에 한 쓰레드 만이 처리하도록 하는 것이다.

- 동기화 선언 방법
  ![](https://velog.velcdn.com/images/sanizzang00/post/527019c5-5fa7-4def-8e51-d8b844bd7437/image.png)

-> 연산이 겹치면 안되는 부분에 사용하면 된다.

#### sleep, notify, notifyAll, wait

![](https://velog.velcdn.com/images/sanizzang00/post/9141f2c0-1fb8-4bb1-923e-75e89bee5c0e/image.png)

yield(): 다른 스레드에게 실행 양보
-> A, B thread가 있을 때 A thread에서 yield()를 호출하면 CPU에게 다른 thread를 실행하라(양보)

### 폴링(Polling)

: 스레드에 플래그를 놓아 값이 준비된다면 플래그를 메인 프로그램에 전달해주는 것
메인 프로그램은 주기적으로 스레드에 값이 준비되었는지 물어보는 작업이 필요
![](https://velog.velcdn.com/images/sanizzang00/post/6d69219d-7471-40c4-b7ef-530a8c42d24a/image.png)

위와 같이 프로그램이 실행 된 이후 스레드에 값이 입력 되었는지 계속 물어보는 작업이다.

Polling을 구현하기 위한 플래그로 finflag 변수를 boolean형으로 추가.
![](https://velog.velcdn.com/images/sanizzang00/post/b6f4aa58-576c-4576-b484-727a734e5110/image.png)
![](https://velog.velcdn.com/images/sanizzang00/post/c166da3b-cc69-4200-8725-49162bd8576f/image.png)

폴링 방식은 쓰레드에 플래그를 두어 쓰레드의 종료 방법을 알 수 있는 매우 간단한 방법이다. 게다가 코딩 작업도 그리 어렵지 않게 구현할 수 있기에 처음 쓰레드를 사용하는 개발자에게는 좋은 방법이다. 하지만, While(true) 문은 항상 위험성이 존재한다. 프로그램을 쉽게 무한 루프에 빠지게 만드는 주요 원인이다. 위와 같은 작업은 우리에게 불필요한 작업을 너무 많이 요구하게 된다.

그리고 매우 낮은 확률로 발생하지만, 뒷부분 쓰레드는 실행이 안될 수도 있다. 앞선 쓰레드가 너무 빨리 종료하게 된다면 Polling 부분에서 더 이상 쓰레드가 남아 있지 않는다고 판단해 프로그램을 종료시키는 경우도 발생한다. 아마 위 프로그램의 입력 계좌번호 수를 크게하고 앞부분의 계좌번호를 짧게 작성하면 앞서 언급한 문제가 발생할 수도 있다. 이는 프로그램에 치명적인 오류를 가져오게 된다.

### 콜백(Callback)

폴링의 문제점 2가지는 다음과 같다.

- 메인 프로그램에서 쓰레드에 작업이 종료되었는지 지속적으로 물어봐야 한다는 점
- 너무 빠른 쓰레드 종료로 메인 프로그램에서 쓰레드가 더 이상 존재하지 않는다고 판단할 수 있다는 점

이를 해결하기 위해 스레드가 직접 메인 클래스에게 자신이 종료 되었다는 사실을 알려주면 된다. 이를 위해서 스레드가 메인 클래스의 메소드를 호출하는 과정이 필요하다. 스레드가 작업이 끝났을 때 자신을 생성한 클래스를 다시 호출하는 방식인데 이를 콜백(Callback)이라고 부른다.

메인 프로그램에 메소드 하나를 새로 생성해야 한다. 이 메소드는 쓰레드에서 사용할 것이다. 그리고 이 메소드가 실행되었다는 것은 쓰레드의 종료를 의미해야한다. 그래야 메인 프로그램에서 쓰레드 종료 여부를 예측할 수 있다.

![](https://velog.velcdn.com/images/sanizzang00/post/8ba4538b-7911-4bc6-aa58-424a450ec64d/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/3e664503-7818-4dbd-872f-7c9952e3e51b/image.png)

폴링방식에서는 입력이 전부 끝난 뒤 폴링 로직이 실행되기 때문에 모든 입력이 전부 끝난 뒤 폴링 로직이 실행되기 때문에, 모든 입력이 끝난 뒤 암호화 된 결과값이 출력된다. 하지만, 콜백 방식에서는 스레드가 끝나게 되고 바로 값을 노출 시키게 되므로, 입력 한 직후 스레드가 실행되고 결과 값도 노출이 바로 나오게 된다.

만일 폴링에서처럼 입력을 전부 받고나서 결과 값의 출력을 원한다면, 메인클래스에 있는 Callback 메소드에 ArrayList나 Queue를 선언하여 값을 저장한 뒤 값을 출력하는 방식을 채택하면 된다.

![](https://velog.velcdn.com/images/sanizzang00/post/d14ff7f6-d543-41bf-a497-83e63e11b694/image.png)

### Callable과 Future 인터페이스에 대한 이해 및 사용법

- Thread와 Runnable의 단점 및 한계

  - 지나치게 저수준의 API(쓰레드의 생성)에 의존함
  - 값의 반환이 불가능
  - 매번 쓰레드 생성과 종료하는 오버헤드가 발생
  - 쓰레드들의 관리가 어려움

- Callable 인터페이스
  기존의 Runnable 인터페이스는 결과를 반환할 수 없다는 한계점이 있었다. 반환값을 얻으려면 공용 메모리나 파이프 등을 사용해야 했는데, 이러한 작업은 상당히 번거롭다. 그래서 Runnable의 발전된 형태로써, Java5에 함께 추가된 제네릭을 사용해 결과를 받을 수 있는 Callable이 추가되었다.

- Future 인터페이스
  Callable 인터페이스의 구현체인 작업(Task) 가용 가능한 쓰레드가 없어서 실행이 미뤄질 수 있고, 작업 시간이 오래 걸릴 수도 있다. 그래서 실행 결과를 바로 받지 못하고 미래의 어느 시점에 얻을 수 있는데, 미래에 완료된 Callable의 반환값을 구하기 위해 사용되는 것이 Future이다. 즉, Future는 비동기 작업을 갖고 있어 미래에 실행결과를 얻도록 도와준다. 이를 위해 비동기 작업의 현재 상태를 확인하고, 기다리며, 결과를 얻는 방법 등을 제공한다.

- get
  - 블로킹 방식으로 결과를 가져옴
  - 타임아웃 설정 가능
- isDone, isCancelled
  - isDone은 작업의 완료 여부, isCancelled는 작업의 취소여부를 반환함
  - 완료 여부를 boolean으로 반환
- cancel
  - 작업을 취소시키며, 취소 여부를 boolean으로 반환함
  - cancle 후에 isDone()는 항상 true를 반환함

### Executors, Executor, ExevutorService 등에 대한 이해 및 사용법

: 쓰레드의 생성과 관리를 위한 쓰레드 풀을 위한 기능들

- Executor 인터페이스
  : 동시에 여러 요청을 처리해야 하는 경우에 매번 새로운 쓰레드를 만드는 것은 비효율적이다. 그래서 쓰레드를 미리 만들어주고 재사용하기위한 쓰레드 풀(Thread Pool)이 등장하게 되었는데, Executor인터페이스는 쓰레드 풀의 구현을 위한 인터페이스이다. 이러한 Executor인터페이스를 간단히 정리하면 다음과 같다.
  - 등록된 작업(Runnable)을 실행하기 위한 인터페이스
  - 작업 등록과 작업 실행 중에서 작업 등록만을 책임짐

쓰레드는 크게 작업의 등록과 실행으로 나누어진다. 그 중에서도 Executor 인터페이스는 인터페이스 분리 원칙(Interface Segregation Principle)에 맞게 등록된 작업을 실행하는 책임만 갖는다. 그래서 전달받은 작업(Runnable)을 실행하는 메소드만 가지고 있다.

Executor 인터페이스는 개발자들이 해당 작업의 실행과 쓰레드의 사용 및 스케줄링 등으로부터 벗어날 수 있도록 도와준다.

- ExecutorService 인터페이스
  ExecutorService는 작업(Runnable, Callable)등록을 위한 인터페이스이다.
  ExecutorService는 Executor를 상속받아서 작업 등록뿐만 아니라 실행을 위한 책임도 갖는다. 그래서 쓰레드 풀은 기본적으로 ExecutorService 인터페이스를 구현한다. 대표적으로 ThreadPoolExecutor가 ExecutorService의 구현체인데, ThreadPoolExecutor 내부에 있는 블록킹 큐에 작업들을 등록해 둔다.

각각의 쓰레드는 작업들을 할당받아 처리하는데, 만약 사용가능한 쓰레드가 없다면 작업은 큐에 대기하게 된다. 그러다가 쓰레드가 작업을 끝내면 다음 작업을 할당받게 되는 것이다. 이러한 ExecutorService가 제공하는 퍼블릭 메소드들은 다음과 같이 분류 가능하다.

- 라이프사이클 관리를 위한 기능들
- 비동기 작업을 위한 기능들
