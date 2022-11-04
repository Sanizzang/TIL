### SQL 중심적인 개발의 문제점?

#### 무한반복, 지루한 코드

CRUD
INSERT INTO ...
UPDATE ...
SELECT ...
DELETE ...
자바 객체를 SQL로...
SQL을 자바 객체로...

```java
public class Memeber {
    private String memeberId;
    private String name;
    // 여기에 tel 정보를 추가하고 싶다면?
}
```

->

```sql
INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES

SELECT MEMBER_ID, NAME, TEL FROM MEMBER M
```

위와 같이 쿼리에 TEL을 각각 다 추가해야한다.
(SQL 의존적인 개발을 피하기 어렵다.)

### 패러다임의 불일치

관계형 DB의 사상과 객체지향의 사상은 완전히 다르다.
관계형 DB는 데이터를 잘 정규화해서 보관하는게 목표,
객체라는 것은 속성과 기능 필드나 메소드 이런게 묶여서 잘 캡슐화하는게 목표

'객체 지향 프로그래밍은 추상화, 캡슐화, 정보은닉, 상속, 다형성 등 시스템의 복잡성을 제어할 수 있는 다양한 장치들을 제공한다.'

### 객체와 관계형 데이터베이스의 차이

1. 상속
   데이터베이스는 상속이 없다고 본다.(비슷한게 있긴 함 -> Table 슈퍼타입 서브타입 관계)
2. 연관관계
   객체 -> 참조(get), 데이터베이스 -> PK,FK(외래키)로 JOIN을 통해
3. 데이터 타입
4. 데이터 식별 방법

객체를 테이블에 맞추어 모델링

계층형 아키텍처
진정한 의미의 계층 분할이 어렵다.

객체답게 모델링 할수록 매핑 작업만 늘어난다.

객체를 자바 컬렉션에 저장하듯이 DB에 저장? -> JPA사용

### JPA?

- Java Persistence API
- 자바 진영의 ORM 기술 표준

### ORM?

- Object-relational mapping (객체 관계 매핑)
- 객체는 객체대로 설계
- 관계형 데이터베이스는 관계형 데이터베이스대로 설계
- ORM 프레임워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재

JPA는 애플리케이션과 JDBC 사이에서 동작

#### JPA 동작 - 저장

JPA에게 Member 객체(Entity)를 넘김
-> JPA가 Member(Entity)를 분석
-> INSERT SQL 생성
-> JDBC API 사용(쿼리를 개발자가 아니라 JPA가 만들어 준다.)
-> 패러다임 불일치 해결

#### JPA 동작 - 조회

JPA에게 find(id) (PK값)만 넘김
-> SELECT SQL 생성
-> JDBC API 사용
-> ResultSet 매핑
-> 패러다임 불일치 해결

JPA가 뭔가?
과거에는 EJB - 엔티티 빈(자바 표준)라는 것이 있었음
이것의 문제는 너무 아마추어적이다. 인터페이스가 너무 많고 동작도 느림
->
하이버네이트(오픈 소스) 개발
->
JPA(자바 표준)
하이버네이트를 거의 복사 붙여넣기해서 만듬
(Hibernate를 다듬어서 다시 재개발)

#### JPA는 표준명세

- JPA는 인터페이스 모음
- JPA 2.1 표준 명세를 구현한 3가지 구현체
- 하이버네이트, EclipseLink, DataNucleus

### JPA를 왜 사용해야 하는가?

- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성
- 유지보수
- 패러다임 불일치 해결
- 성능
- 데이터 접근 추상화와 벤더 독립성
- 표준

#### 생산성 - JPA와 CRUD

- 저장: jpa.persist(member)
- 조회: Member member = jpa.find(memberId)
- 수정: member.setName("변경할 이름")
- 삭제: jpa.remove(member)

#### 유지보수 - 기존: 필드 변경시 모든 SQL 수정

- JPA: 필드만 추가하면 됨, SQL은 JPA가 처리

#### JPA와 패러다임의 불일치 해결

1. JPA와 상속
2. JPA와 연관관계
3. JPA와 객체 그래프 탐색
4. JPA와 비교하기

### JPA의 성능 최적화 기능

1. 1차 캐시와 동일성(identity) 보장
2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
3. 지연 로딩(Lazy Loading)

JPA를 잘 사용하면 일반 SQL을 사용하는 것보다 성능을 끌어 올릴 수 있다.
-> JPA가 중간에서 위와 같은 성능 최적화를 해줌.

#### 1차 캐시와 동일성 보장

1. 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
2. DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장

```java
String memberId = "100";
Member m1 = jpa.find(Member.class, memberId); //SQL
Member m2 = jpa.find(Member.class, memberId); //캐시

println(m1 == m2) //true
// SQL 1번만 실행
```

#### 트랜잭션을 지원하는 쓰기 지연 - INSERT

1. 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
2. JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

```java
transaction.begin(); // [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
// 여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

// 커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

#### 지연 로딩과 즉시 로딩

- 지연 로딩: 객체가 실제 사용될 때 로딩

```java
Member member = memberDAO.find(memberId); // -> SELECT * FROM MEMBER
Team team = member.getTeam();
String teamName = team.getName(); // -> SELECT * FROM TEAM
```

- 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

```java
Member member = memberDAO.find(memberId); // -> SELECT M.*, T.* FROM MEMBER JOIN TEAM...
Team team = member.getTeam();
String teamName = team.getName();
```

옵션을 통해 위의 로딩을 설정할 수 있다.

ORM은 객체와 RDB 두 기둥위에 있는 기술
