하나의 마이크로서비스를 하나 이상의 인스턴스에서 기동을 시켰을 때 클라이언트 요청이 여러개가 들어왔을 때 그것을 부하분산 처리하기 위해서 우리가 여러개의 인스턴스를 띄울 수 있다.(로드밸런싱) 마이크로서비스를 기동을 할 때, 우리가 서비스 port를 지정하게 되는데, 서비스 port를 0번으로 지정을 하게되면, 랜덤 port가 지정이 되서 여러개의 인스턴스를 실행한다 하더라도 충돌이 발생하지 않는다.

예를 들어, User Service가 Order Service의 어떤 요청을 해서 자기가 필요한 데이터를 가져가고 주문도 한다고 가정했을 때, 2가지 이상의 Order Service를 기동했다고 가정 각각 데이터베이스를 가지고 있다. 이렇게 되면 Order 데이터도 분산 저장되있을 수 밖에 없다. 우리가 사용하고 있는 order라는 마이크로서비스는 독립적인 데이터베이스를 가질 수 있도록 구성을 해놨다.

order-service 인스턴스가 기동이될 때 H2라는 내장DB가 같이 기동이 된다. 각각의 데이터베이스가 각각의 인스턴스에 사용되고 있다.
-> 하나의 주문에 대해서 나눠서 저장되어 있는 주문 데이터 값을 동기화를 어떻게 처리할 것인지 문제가 나온다.

예를 들어 order service를 2가지를 기동 데이터베이스도 2가지로 분산되서 저장

### 데이터 동기화를 어떻게 할 것인가?

#### 1.하나의 Database 사용

물리적으로 떨어져 있는 인스턴스를 하나의 데이터베이스에 저장하기 위해서는 트랜잭션 관리를 잘 해줘야한다.(동시성)

#### 2. Database간의 동기화

각각 1번과 2번의 데이터베이스에서 필요한 데이터 값을 하나씩 전달하는 방법이 아니라 중간에 메시지 큐잉이라는 서버를 통해서(Apache Kafka) 한 쪽에서 발생했던 데이터를 메시지 큐잉 서버에 전달, 메시지 큐잉서버에 변경된 데이터가 있으면 알려달라는 구독신청 -> 두가지 데이터베이스가 동기화가 가능

#### 3. Kafka Connector + DB

하나의 데이터베이스 사용과 데이터 베이스 동기화의 복합 예제

메시지 큐잉 서버를 사용하고 데이터베이스도 하나를 사용
첫 번째 order service 두 번째 order service에서 발생한 데이터를 메시지 큐잉 서버에 메시지를 보내게된다. 메시지 큐잉 서버 자체는 이런 처리를 하기 위해 특화되어 있는 시스템이다 보니 메시지가 전달되면 순차적으로 해석해서 이 메시지를 사용하고자 하는 곳에 사용할 수 있게끔 뿌려주는 중간 매개체(미들웨어) 역할을 한다. 아무리 많은 역할을 한다 하더라도 1초안에 수만건의 데이터를 처리할 수 있는 기술이기 때문에 동시성에 대한 문제라던가 시간적인 배분에 대한 것들은 충분히 해결할 수 있는 능력이 있다.
그리고 메시지 큐잉에 전달된 데이터를 하나의 단일 데이터베이스에 저장한다고 하면 각각의 order service가 자신이 필요한 데이터를 가져갔을때 동일한 데이터베이스를 사용하기 때문에 데이터 간에 일치하지 않는 문제는 해결할 수 있다.

## 데이터 동기화를 위한 Kafka 활용 1

### Apache Kafka

- Apache Software Foundation의 Scalar 언어로 된 오픈 소스 메시지 브로커 프로젝트
  - Open Source Message Broker Project
- 링크드인(Linked-in)에서 개발, 2011년 오픈 소스화
  - 2014년 11월 링크드인에서 Kafka를 개발하던 엔지니어들이 Kafka 개발에 집중하기 위해 Confluent라는 회사 창립
- 실시간 데이터 피드를 관리하기 위해 통일된 높은 처리량, 낮은 지연 시간을 지닌 플랫폼 제공
- Apple, Netflix, Shopify, Yelp, Kakao, New York Times, Cisco, Ebay, Paypal, Hyperledger Fabric, Uber, Salesforce.com 등이 사용

- 메시지 브로커?
  : 특정한 리소스에서 다른쪽의 리소스(서비스, 시스템)으로 메시지를 전달할 떄 사용되는 서버

전산시스템에서의 메시지는 일반적인 텍스트도 가능하지만 JSON, XML, Object 등의 다양한 데이터들도 가능. 그런 데이터를 보내는 쪽과 받는 쪽으로 구분을 시킴

- 모든 시스템으로 데이터를 실시간으로 전송하여 처리할 수 있는 시스템
- 데이터가 많아지더라도 확장이 용이한 시스템

보내는 쪽과 받는 쪽이 누가 보냈고 누가 보냈는지 신경쓰지 않는 상태에서
메시지를 받는게 가능해짐

- 메시지를 보내는 쪽: Producer
- 메시지를 받는 쪽: Consumer

- Producer / Consumer 분리
- 메시지를 여러 Consumer에게 허용
- 높은 처리량을 위한 메시지 최적화
- Scale-out 가능
- Eco-system

### Kafka Broker

- 실행 된 Kafka 애플리케이션 서버
- 3대 이상의 Broker Cluster 구성
- Zookeeper 연동
  - 역할: 메타데이터 (Broker ID, Controller ID 등) 저장
  - Controller 정보 저장
- n개 Broker 중 1대는 Controller 기능 수행
  - Controller 역할
    - 각 Broker에게 담당 파티션 할당 수행
    - Broker 정상 동작 모니터링 관리

Kafka는 기본적으로 메시지를 보내면 그 데이터는 topic이라는 곳에 저장이된다.
Topic은 어떻게 생성을 하냐먼 임의로 자유롭게 생성할 수 있다.

#### Kafka 서버 기동 - Windows

- Windows에서는 Kafka 실행 명령어는 $KAFKA_HOME\bin\windows 폴더에 저장되어 있음

```
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

#### Kafka Producer/Consumer 테스트

- 메시지 생산

```
$KAFKA_HOME\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic quickstart-events
```

- 메시지 소비

```
$KAFKA_HOME\bin\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning
```

### Kafka Connect

- Kafka Connect를 통해 Data를 Import/Export 가능
- 코드 없이 Configuration으로 데이터를 이동
- Standalone mode, Distribution mode 지원

  - RESTful API 통해 지원
  - Stream 또는 Batch 형태로 데이터 정송 가능
  - 커스텀 Connector를 통한 다양한 Plugin 제공(File, S3, Hive, Mysql, etc ...)

- 데이터를 가져오는 쪽: Kafka Connect Source
- 데이터를 보내는 쪽: Kafka Connect Sink
