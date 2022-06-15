### FOREIGN KEY

한 테이블을 다른 테이블과 연결해주는 역할을 한다.
외래 키가 설정된 테이블에 레코드를 입력하면, 기준이 되는 테이블의 내용을 참조해서 레코드가 입력된다.
즉, FOREIGN KEY 제약 조건은 하나의 테이블을 다른 테이블에 의존하게 만든다.

FOREIGN KEY 제약 조건을 설정할 때 참조되는 테이블의 필드는 반드시 UNIQUE나 PRIMARY KEY 제약 조건이 설정되어 있어야 한다.

### CREATE 문으로 FOREIGN KEY 설정

```sql
CREATE TABLE 테이블이름
(
    필드이름 필드타입,
    ...,
    [CONSTRAINT 제약조건이름]
    FOREIGN KEY (필드이름)
    REFERENCES 테이블이름 (필드이름)
)
```

### ALTER 문으로 FOREIGN KEY 설정

```sql
ALTER TABLE 테이블이름
ADD [CONSTRAINT 제약조건이름]
FOREIGN KEY (필드이름)
REFERENCES 테이블이름 (필드이름)
```

### ON DELETE, ON UPDATE

FOREIGN KEY 제약 조건에 의해 참조되는 테이블에서 수정이나 삭제가 발생하면, 참조하고 있는 테이블의 데이터도 같이 영향을 받는다.

1. ON DELETE
   참조되는 테이블의 값이 삭제될 경우의 동작
2. ON UPDATE
   참조되는 테이블의 값이 수정될 경우의 동작

설정할 수 있는 동작

1. CASCADE: 참조되는 테이블에서 데이터를 삭제하거나 수정하면, 참조하는 테이블에서도 삭제와 수정이 같이 이루어진다.
2. SET NULL: 참조되는 테이블에서 데이터를 삭제하거나 수정하면, 참조하는 테이블의 데이터는 NULL로 변경된다.
3. NO ACTION: 참조되는 테이블에서 데이터를 삭제하거나 수정해도, 참조하는 테이블의 데이터는 변경되지 않는다.
4. SET DEFAULT: 참조되는 테이블에서 데이터를 삭제하거나 수정하면, 테이블의 데이터는 필드의 기본 값으로 설정된다.
5. RESTRICT: 참조하는 테이블에 데이터가 남아 있으면, 참조되는 테이블의 데이터를 삭제하거나 수정할 수 없다.
