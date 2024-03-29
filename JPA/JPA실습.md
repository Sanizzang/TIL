### JPA 설정하기 - persistence.xml

- JPA 설정 파일
- /META/-INF/persistence.xml 위치
- persistence-unit name으로 이름 지정
- javax.persistence로 시작: JPA 표준 속성
- hibernate로 시작: 하이버네이트 전용 속성

### 데이터베이스 방언

- JPA는 특정 데이터베이스에 종속 X
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
  - 가변 문자: MySQL은 VARCHAR, Oracle은 VARCHAR2
  - 문자열을 자르는 함수: SQL 표준은 SUBSTRING(), Oracle은 SUBSTR()
  - 페이징: MySQL은 LIMIT, Oracle은 ROWNUM
- 방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능

- hibernate.dialect 속성에 지정
- H2: org.hibernate.dialect.H2Dialect
- Oracle 10g: org.hibernate.dialect.Oracle10gDialect
- MySQL: org.hibernate.dialect.MySQL5InnoDBDialect
- 하이버 네이트는 40가지 이상의 데이터베이스 방언 지원
  -> 현업에서 쓸 수 있는 모든 방언이 있다.

### JPA 구동방식

Persistence
-> 1. 설정 정보 조회(META-INF/persistence.xml)
-> 2. 생성 (EntityManagerFactory)
-> 3. 생성 (EntityManager, EntityManager, EntityManager)

### 객체와 테이블을 생성하고 매핑하기

@Entity: JPA가 관리할 객체

@Id: 데이터베이스 PK와 매핑

```java
@Entity
@Table(name="USER")
public class Member {

    @Id
    private Long id;

    // column 명 변환가능
    @Column(name = "username")
    private String name;

    // Getter, Setter
}
```

```sql
create table Member (
    id bigint not null,
    name varchar(255),
    primary key (id)
)
```

```java
// 데이터베이스 연결
EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");

EntityManager em = emf.createEntityManager();

// JPA에서 트랜잭션은 매우 중요하다.
// -> 꼭 트랜잭션 안에서 작업해야한다.
EntityTransaction tx = em.getTransaction();
tx.begin();

try{
    Member member = new Member();

    member.setId(1L);
    member.setName("HelloA);

    em.persist(member);

    tx.commit();
} catch(Exception e) {
    tx.rollback();
}
finally {
    em.close();
}

emf.close();
```

수정

```java
Member member = new Member();

    member.setId(1L);
    member.setName("HelloA);

    tx.commit();
```

-> persist를 하지 않아도 수정이 된다.(자바 객체 값만 바꿔도 됨)
-> JPA관련 메소드(find)를 사용하게 되면 해당 객체를 JPA가 관리하게 된다.
-> JPA가 객체의 변경사항을 저장해 두었다가 commit 당시에 업데이트를 해주게 된다.

### 주의

- 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에서 공유
- 엔티티 매니저는 쓰레드간에 공유X (사용하고 버려야 한다).
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

### JPQL 소개

- 가장 단순한 조회 방법
  - EntityManager.find()
  - 객체 그래프 탐색(a.getB().getC())
- 나이가 18살 이상인 회원을 모두 검색하고 싶다면?

- 전체 회원 조회

```java
// 잘보면 약간 다르다. JPA 입장에서는 Table대상으로 코드를 짜지 않는다. -> 객체를 대상으로 함(객체지향 Query)
List<Member> result = em.createQuery("select m from Member as m", Member.class)
                    .getResultList();
```

### JPQL

- JPA를 사용하면 엔티티 객체를 중심으로 개발
- 문제는 검색 쿼리
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- JPQL을 한마디로 정의하면 객체 지향 SQL
- JPQL은 뒤에서 아주 자세히 다룸
