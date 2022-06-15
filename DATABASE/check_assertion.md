### CHECK

- 입력 값이 조건에 맞지 않으면 DB에서 받지 않는다. 즉 오료를 일으킨다.
- 입력 값의 범위를 지정할 수 있다.

즉 CHECK은 입력할 수 있는 값의 범위를 설정해 주는 것이다.

> 이미 들어가 있는 데이터가 조건에 위배되면 적용이 안된다.

```sql
ALTER TABLE [테이블명] ADD CONSTRAINT [제약조건명] [제약조건](범위)
```

### 주장(ASSERTION)

릴레이션에 변경이 발생 시 항상 참(TRUE)가 되는 조건식을 정의한다.
조건식은 일반적으로 진단화 연산(EXIST, Not EXIST)를 사용한다.
Create assertion [Name] check [Condition]
조건(condition): 조건은 항상 참(TRUE)이 되어야 함

```sql
CREATE ASSERTION Salary_limit
CHECK
(NOT EXISTS
(
    SELECT *
    FROM EMP
    WHERE SAL > 5000
)
);
```
