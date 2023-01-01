## 경로 표현식

---

- .(점)을 찍어 객체 그래프를 탐색하는 것

```sql
select m.username  -> 상태 필드
  from Member m
  join m.team t    -> 단일 값 연관 필드
  join m.orders o  -> 컬렉션 값 연관 필드
where t.name = '팀A'
```

### 경로 표현식 용어 정리

- 상태 필드(state field): 단순히 값을 저장하기 위한 필드 (ex: m.username)
- 연관 필드(association field): 연관관계를 위한 필드
  - 단일 값 연관 필드:
    @ManyToOne, @OneToOne, 대상이 엔티티(ex: m.team)
  - 컬렉션 값 연관 필드:
    @OneToMany, @ManyToMany, 대상이 컬렉션(ex: m.orders)

### 경로 표현식 특징

- 상태 필드(state field): 경로 탐색의 끝, 탐색X
- 단일 값 연관 경로: 묵시적 내부 조인(inner join) 발생, 탐색O
- 컬렉션 값 연관 경로: 묵시적 내부 조인 발생, 탐색X
  - FROM절에서 명시적인 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능

### 상태 필드 경로 탐색

- JPQL: select m.username, m.age from Member m

- SQL: select m.username, m

### 단일 값 연관 경로 탐색

- JPQL: select o.member from Order o

- SQL:
  select m.\*
  from Orders o
  inner join Member m on o.member_id = m.id

### 명시적 조인, 묵시적 조인

- 명시적 조인: join 키워드 직접 사용
  - select m from Member m join m.team t
- 묵시적 조인: 경로 표현식에 의해 묵시적으로 SQL 조인 발생(내부 조인만 가능)
  - select m.team from Member m

### 경로 탐색을 사용한 묵시적 조인 시 주의사항

- 항상 내부 조인
- 컬렉션은 경로 탐색의 끝, 명시적 조인을 통해 별칭을 얻어야함
- 경로 탐색은 주로 SELECT, WHERE절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM(JOIN) 절에 영향을 줌

### 실무 조언

- 가급적 묵시적 조인 대신에 명시적 조인 사용
- 조인은 SQL 튜닝에 중요 포인트
- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움

## 페치 조인 1 - 기본

---

### 페치 조인(fetch join)

- SQL 조인 종류X
- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능
- join fetch 명령어 사용
- 페치 조인 ::= [LEFT [OUTER] | INNER] JOIN FETCH 조인경로

### 엔티티 페치 조인

- 회원을 조회하면서 연관된 팀도 함께 조회(SQL 한 번에)
- SQL을 보면 회원 뿐만 아니라 팀(T.\*)도 함께 SELECT
- [JPQL]
  select m from Member m join fetch m.team
- [SQL]
  SELECT M._, T._ FROM MEMBER M
  INNER JOIN TEAM T ON M.TEAM_ID=T.ID

```java
String jpql = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(jpql, Member.class)
                            .getResultList();

for(Member member: members) {
    // 페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X
    System.out.println("username = " + member.getUsername() + ", " +
                        "teamName = " + member.getTeam().name());
}
```

### 페치 조인과 DISTINCT

- SQL의 DISTINCT는 중복된 결과를 제거하는 명령
- JPQL의 DISTINCT 2가지 기능 제공
  - 1. SQL에 DISTINCT를 추가
  - 2. 애플리케이션에서 엔티티 중복 제거
- DISTINCT가 추가로 애플리케이션에서 중복 제거시도
- 같은 식별자를 가진 Team 엔티티 제거

### 페치 조인과 일반 조인의 차이

- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념

## 패치 조인 - 한계

---

### 페치 조인의 특징과 한계

- 페치 조인 대상에는 별칭을 줄 수 없다.
  - 하이버네이트는 가능, 가급적 사용X
- 둘 이상의 컬렉션은 조인할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
  - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
  - 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)
