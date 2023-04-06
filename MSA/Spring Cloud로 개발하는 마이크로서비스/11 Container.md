컨테이너 가상화 기반으로 애플리케이션을 구성하게 되면 local PC 뿐만 아니라 remote 또는 Public Cloud, Private Cloud에다가
그대로 환경을 이전시켜서 사용할 수 있다.

### Virtualization

- 물리적인 컴퓨터 리소스를 다른 시스템이나 애플리케이션에서 사용할 수 있도록 제공

  - 플랫폼 가상화
  - 리소스 가상화

- 하이퍼바이저 (Hypervisor)
  - Virtual Machine Manager (VMM)
  - 다수의 운영체제를 동시에 실행하기 위한 논리적 플랫폼
  - Type1: Native or Bare-metal
    : Hard ware에서 직접 Hypervisor를 설치해 줌으로써 가상화를 운영할 수 있는 방식
  - Type2: Hosted
    : Hard ware에 운영체제를 설치해 그 위에 Hypervisor 기능을할 수 있는 소프트웨어를 설치해 줌으로써 가상화를 사용할 수 있는 방법

### Container Virtualization

- OS Virtualization

  - Host OS(PC를 처음 기동했을 때 사용되어지는 운영체제) 위에 Guest OS(Host 운영체제 위에 가상화로 설치되어있는 운영체제) 전체를 가상화
  - VMWare, VirtualBox (Hypervisor)
  - 자유도가 높으나, 시스템에 부하가 많고 느려짐
    : 하드웨어 위에 Host PC 위에 Guest OS를 여러개를 두는 경우 시스템 부하가 많이 걸릴 수 밖에 없다.

- Container Virtualization
  - Host OS가 가진 리소스를 적게 사용하며, 필요한 프로세스 실행
  - 최소한의 라이브러리와 도구만 포함
  - Container의 생성 속도 빠름

```
Hypervisor가 올라와야할 위치에 컨테이너 가상화 솔루션(Docker Engine)을 대신 실행할 수 있다.
Container 가상화를 사용하면 중복적인 코드가 있다면 그것을 사용하지 않을 수 있다.
Guest OS 자체가 설치되어지는 것이아니라 컨테이너를 실행할 수 있는 소프트웨어, 가상화를 실행하기 위한 필요한 것들만 실행해주면 된다.
나머지는 Host OS가 가지고 있는 리소스와 Docker Engine이 가지고 있는 리소스를 공유하고 사용함으로써
가상으로 만들어진 소프트웨어 운영체제같은 것들을 실행할 때 최소한의 내용만을 가지고 실행할 수 있기 떄문에
OS 가상화보다는 적은 리소스를 사용하고 속도도 빠르게 사용할 수 있다는 장점이 있다.
```

### Container Image

- Container 실행에 필요한 설정 값
  - 상태값x, Immutable
- Image를 가지고 실체화 -> Container

ex) Ubuntu 이미지라는 것은 Ubuntu 이미지를 실행하기 위한 파일과 실행 명령어 포트정도들이다. 컨테이너를 실행하기 위한 모든정보를 가지고 있기 떄문에
의존성 파일을 컴파일하거나 설치할 필요가 없는 상태, 이미지만 가지고 있으면 하나의 소프트웨어라던가 운영체제를 바로 실행할 수 있는 상태

새로운 서버가 실행이 되면 미리 만들어 놓은 이미지를 다운로드 받고 해당하는 이미지를 통해서 해당하는 Container 생성만 해주면 된다.
한 서버에 여러개의 컨테이너를 실행할 수 있다.

- Public Registry(Docker Hub)
  이미지를 저장할 수 있는 저장소
  간단한 서버를 기동해서 Private Registry를 운영할 수 도 있다.

이러한 이미지를 사용할 수 있는 컨테이너 서버 자체를 Docker Host라고 얘기를 하고
Docker Host에서는 Host에서 실행할 수 있는 Container가 저장될 수 있는 자체 Local Repository를 가지고있다.

Local Repository에 저장되어지는 것은 Public Registry 던가 Private Registry에 들어가 있던 이미지를 다운로드 받아서
Local에 저장할 수 있게된다.

Local에 저장했던 이미지를 가지고 필요한 컨테이너를 생성할 수 있다.

Container를 실행하기 위해서 필요한 Command가 create, start, run이라는 종류의 커맨드가 있다.

- create: 컨테이너 생성
- start: 생성된 컨테이너를 실행해주는 명령어
- run: create와 start가 복합되어 있는거라고 보면된다. (생성, 시작 다 해줌)
  Local Repository에 저장되어 있지 않은 Image가 있다고 하면 그 Image를 Pull(다운로드) 받고 create도 해주고 start도 해주는 것

이렇게 Docker Host안에 생성되어진 실체화된 Container 들을 외부에서 Client들이 사용하기 위해서 적절한 Port가 공개되어 있다고 하면
Client에서 이게 독립적으로 운영될 수 있는 서비스 처럼 실제로 실행을 해서 사용할 수 있다.

### Dockerfile

- Docker Image를 생성하기 위한 스크립트 파일
- 자체 DSL(Domain-Specific language) 언어 사용 -> 이미지 생성과정 기술

```dsl
FROM myslq:5.7

ENV MYSQL_ALLOW_EMPTY_PASSWORD true
ENV MYSQL_DATABASE mydb

ADD ../db_mount /var/lib/mysql

EXPOSE 3306

CMD ["mysqld"]
```

### Docker 실행

```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

- 옵션
  - -d: detached mode 흔히 말하는 백그라운드 모드
  - -p: 호스트와 컨테이너의 포트를 연결 (포워딩)
  - -v: 호스트와 컨테이너의 디렉토리를 연결 (마운트)
  - -e: 컨테이너 내에서 사용할 환경변수 설정
  - --name: 컨테이너 이름 설정
  - --rm: 프로세스 종료시 컨테이너 자동 제거
  - -it: -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
  - --link: 컨테이너 연결[컨테이너명:별칭]

```
docker run ubuntu:16.04
```

위의 명령거 같은 경우는 컨테이너가 실행됐다가 바로 종료가 된다.
ubuntu같은 경우는 프로세스를 계속 진행할 만한 코드를 가지고 있지 않다.
따라서 프로세스가 실행이 됐다가 바로 종료가 된다.

ubuntu 같은 경우는 이 자체가 사용된다기 보다는 이러한 이미지가 base 이미지가 되서
추가적인 서비스 같은 것들을 만들 떄 사용이 된다.

하나의 이미지는 하나의 파일로 되어 있는 것이 아니라 필요한 정보에 따라서
Layer로 구분되어 있고 그런 Layer가 쪼개져서 다운로드 된다.

### Docker 실행

```
docker run -d -p 3306: 3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7
```

```
docker exec -it mysql bash
```

exec: 실행중인 컨테이너에 추가적인 작업을 하고자 할때

### Docker 이미지 생성과 Public registry에 Push

```
FROM openjdk:8-jdk-alpine

// tmp: 도커 엔진이 가지고 있는 가상의 디렉토리
VOLUME /tmp

// 현재 폴더의 target/users-ws-0.1.jar 파일을 컨테이너에 users-service.jar로 바꾸겠다.
COPY target/users-ws-0.1.jar users-service.jar

// 도커 컨테이너가 제일 마지막에 어떠한 절차로 어떠한 명령어를 가지고 도커를 실행할 것인지 실행 커맨드를 명시해 놓은 것
ENTRYPOINT ["java",
"-Djava.security.egd=file:/dev/./urandow",
"-jar",
"users-service.jar"]
```

```
// 도커 이미지 빌드
// . : 현재 디렉토리에서 Dockerfile을 찾아 Docker 이미지를 빌드
$ docker build -t edowon0623/user-service:1.0 .
// Docker Hub나 다른 Docker 레지스트리에 로그인한 후에 로컬 머신에 빌드한 Docker 이미지를 푸시하는 명령어
$ docker push edowon0623/user-service:1.0
$ docker pull edowon0623/user-service:1.0
```
