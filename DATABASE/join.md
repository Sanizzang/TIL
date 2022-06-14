#### 참고

https://advenoh.tistory.com/23
http://www.tcpschool.com/mysql/mysql_multipleTable_join

### JOIN

JOIN은 데이터베이스 내의 여러 테이블에서 가져온 레코드를 조합하여 하나의 테이블이나 결과 집합으로 표현해 준다.
이러한 JOIN은 보통 SELECT 문과 함께 자주 사용된다.

### JOIN의 종류

- 내부 조인(INNER JOIN)

  - 교차 조인(CROSS JOIN - CARTESIN JOIN)
  - 등가/동등/동일 조인(EQUI JOIN)
  - 비등가 조인(NON-EQUI JOIN)
  - 자연 조인(NATURAL JOIN)

- 외부 조인(OUTER JOIN)

  - 완전 외부 조인(FULL OUTER JOIN)
  - 왼쪽 (LEFT OUTER)
  - 오른쪽 (RIGHT OUTER)

- 셀프 조인 (SELF JOIN)
- 안티 조인 (ANTI JOIN)
- 세미 조인 (SEMI JOIN)

### 내부 조인

#### 교차 조인 (CROSS JOIN-CARTESIN JOIN)

교차 조인은 두 테이블의 카티션 프로덕트(곱집합)를 한 결과이다. 특별한 조건없이 테이블 A의 각 행과 테이블 B의 각 행을 다 조합한 결과이다.

조인 SQL 구문은 두 가지 표현법으로 만들 수 있다. 구체적인 SQL 구문이 있는 명시적 표현법과 암묵적인 표현 방식이 있다.

```sql
# 명시적 표현법 (explicit notation)

SELECT \*
FROM employees
CROSS JOIN dept_emp;

# 암묵적 표현법 (implicit notation)

SELECT \*
FROM employees, dept_emp;
```

#### 내부 조인 (INNER JOIN)

내부 조인은 가장 많이 사용되는 조인 구문중에 하나이다. 내부 조인은 조인 조건문에 따라 2개의 테이블(A, B)의 컬럼을 합쳐 새로운 테이블을 생성한다. 즉, 교차조인을 한 결과에 조인 조건물을 충족시키는 레코드를 반환한다고 생각하면 된다.
![](https://velog.velcdn.com/images/sanizzang00/post/917b023a-0d71-44bf-a0ff-33f37dde4d95/image.png)

```sql
# 명시적 표현법 (explicit notation)

SELECT \*
FROM employees
INNER JOIN dept_emp
ON employees.emp_no = dept_emp.emp_no;

# 암묵적 표현법 (implicit notation)

SELECT \*
FROM employees, dept_emp
WHERE employees.emp_no = dept_emp.emp_no;
```

#### 등가 조인 (EQUI JOIN)

등가 조인은 비교기반 조인의 특정 유형으로 동등비교(=)를 사용하는 조인이다.

#### 비등가 조인 (NON-EQUI JOIN)

비등가 조인은 동등비교(=)를 사용하지 않는 조인으로 조건문이 크거나 작거나 같지 않은 비교등을 사용하면 비등가 조인이라고 한다.

#### 자연 조인 (NATURAL JOIN)

자연 조인은 동등 조인의 한

### 외부 조인 (OUTER JOIN)

내부 조인의 경우에는 공통 컬럼명 기반으로 결과 집합을 생성한다 반면에 외부 조인은 조건문에 만족하지 않는 행도 표시해주는 조인이다.
그래서, 조인을 했을 때 한쪽의 테이블에 데이터가 없어도 조인 결과에 포함시키는 조인이다.

#### 왼쪽 외부 조인 (LEFT OUTER JOIN)

왼쪽 외부 조인은 테이블 A의 모든 데이터와 테이블 B와 매칭이 되는 레코드를 포함하는 조인이다.
![](https://velog.velcdn.com/images/sanizzang00/post/3582de9e-86a9-4802-88c2-2142551c6d68/image.png)

```sql
왼쪽 외부 조인 (LEFT OUTER JOIN)

# 명시적 표현법 (explicit notation)
SELECT *
FROM employees
  LEFT OUTER JOIN departments
    ON employees.dept_no = departments.dept_no;

```

#### 완전 외부 조인 (FULL OUTER JOIN)

![](https://velog.velcdn.com/images/sanizzang00/post/9e1733c9-619b-4d25-a3ae-d6045993bc08/image.png)
