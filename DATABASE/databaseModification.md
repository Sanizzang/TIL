### INSERT

#### 테이블에 레코드 추가

INSERT INTO 문과 함께 VALUES절을 사용하여 해당 테이블에 새로운 레코드를 추가할 수 있다.

```sql
1. INSERT INTO 테이블이름(필드이름1, 필드이름2, 필드이름3, ...)
   VALUES (데이터값1, 데이터값2, 데이터값3, ...)
2. INSERT INTO 테이블이름
   VALUES (데이터값1, 데이터값2, 데이터값3, ...)
```

또한, 두 번째 문법처럼 필드의 이름을 생략할 수 있으며, 이 경우에는 데이터베이스의 스키마와 같은 순서대로 필드의 값이 자동 대입된다.

생략할 수 있는 필드

1. NULL을 저장할 수 있도록 설정된 필드
2. DEFAULT 제약조건이 설정된 필드
3. AUTO_INCREMENT 키워드가 설정된 필드

### UPDATE

```sql
UPDATE 테이블이름
SET 필드이름1=데이터값1, 필드이름2=데이터값2, ...
WHERE 필드이름=데이터값
```

UPDATE문은 해당 테이블에서 WHERE절의 조건을 만족하는 레코드의 값만을 수정한다.

### DELETE

```sql
DELETE FROM 테이블이름
WHERE 필드이름=데이터값
```

DELETE 문은 해당 테이블에서 WHERE 절의 조건을 만족하는 레코드만을 삭제한다.
즉, 테이블에서 명시된 필드와, 그 값이 일치하는 레코드만을 삭제해 준다.