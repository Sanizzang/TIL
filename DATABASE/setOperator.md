### 집합 연산자

두 개 이상의 테이블에서 조인을 사용하지 않고 연관된 데이터를 조회하는 방법 중에 또 다른 방법이 있는데 집합연산자를 사용하는 방법이다.
집합 연산자는 여러 개의 질의의 결과를 연결하여 하나로 결합하는 방식을 사용한다.
집합 연산자를 사용하기 위해서는 SELECT 절의 컬럼 수가 동일하고, 동일 위치에 존재하는 컬럼의 데이터 타입이 상호 호환 가능해야 한다.

### 집합 연산자의 종류

1. UNION
   : 여러개의 SQL문의 결과에 대한 합집합으로 결과에서 모든 중복된 행은 하나의 행으로 출력
2. UNION ALL
   : 여러개의 SQL문의 결과에 대한 합집합으로 중복된 행도 그대로 결과로 표시
3. INTERSECT
   : 여러 개의 SQL문의 결과에 대한 교집합으로 중복된 행은 하나의 행으로 출력
4. EXCEPT
   : 앞의 SQL 문의 결과에서 뒤의 SQL 문의 결과에 대한 차집합으로 중복된 행은 하나의 행으로 출력

```sql
SELECT 컬럼명1, 컬럼명2
FROM 테이블명1
[WHERE 조건식]
[GROUP BY]
[HAVING]
집합 연산자
SELECT 컬럼명1, 컬럼명2
FROM 테이블명2
[WHERE 조건식]
[GROUP BY]
[HAVING]
[ORDER BY]
```
