### 데이터베이스(DataBase)란?

데이터베이스(DB:database)는 통합하여 관리되는 데이터의 집합체를 의미한다.

이는 중복된 데이터를 없애고, 자료를 구조화하여, 효율적인 처리를 할 수 있도록 관리된다.
이러한 데이터베이스는 응용 프로그램과는 다른 별도의 미들웨어에 의해 관리된다. 데이터베이스를 관리하는 이런한 미들웨어를 데이터베이스 관리 시스템(DBMS: Database Management System)이라고 한다.

### 데이터베이스의 특징

1. 사용자의 질의에 대하여 즉각적인 처리와 응답이 이루어진다.
2. 생성, 수정, 삭제를 통하여 항상 최신의 데이터를 유지한다.
3. 사용자들이 원하는 데이터를 동시에 공유할 수 있다.
4. 사용자가 원하는 데이터를 주소가 아닌 내용에 따라 참조할 수 있다.
5. 응용프로그램과 데이터베이스는 독립되어 있으므로, 데이터의 논리적 구조와 응용프로그램은 별개로 동작된다.

### SQL

SQL(Structured Query Language)은 데이터베이스에서 데이터를 정의, 조작, 제어하기 위해 사용하는 언어이다.
따라서 SQL 구문도 위의 목적에 크게 세 가지로 구분할 수 있다.

1. DDL(Data Definition Language)
   : 데이터베이스나 테이블 등을 생성, 삭제하거나 그 구조를 변경하기 위한 명령어
   [CREATE, ALTER, DROP]
2. DML(Data Manipulation Language)
   : 데이터베이스에 저장된 데이터를 처리하거나 조회, 검색하기 위한 명령어
   [INSERT, UPDATE, DELETE, SELECT]
3. DCL(Data Control Language)
   : 데이터베이스에 저장된 데이터를 관리하기 위하여 데이터의 보안성 및 무결성 등을 제어하기 위한 명령어
   [GRANT, REVOKE]

### 제약 조건(constraint)

1. NOT NULL: 해당 필드는 NULL 값을 저장할 수 없게 된다.
2. UNIQUE: 해당 필드는 서로 다른 값을 가져야만 한다.
3. PRIMARY KEY: 해당 필드가 NOT NULL과 UNIQUE 제약 조건의 특징을 모두 가지게 된다.
4. FOREIGN KEY: 하나의 테이블을 다른 테이블에 의존하게 만든다.
5. DEFAULT: 해당 필드의 기본 값을 설정한다.

### Selection in SQL

![](https://velog.velcdn.com/images/sanizzang00/post/d65f3c83-8901-4968-9632-be3017c4e76a/image.png)

### 흐름 제어

#### CASE

CASE 연산자는 값을 서로 비교하거나, 표현식의 논리값에 따라 다른 값을 반환한다.
![](https://velog.velcdn.com/images/sanizzang00/post/c6722daa-26c2-46ea-a882-dc6d4d2a5307/image.png)
첫 번째 CASE 문법에서는 value와 compare_value값이 같으면, THEN 절의 result 값을 반환한다.
만약 서로 값이 같지 않으면, ELSE 절의 result 값을 반환한다.
이때 ELSE 절이 없으면, NULL을 반환한다.

두 번째 CASE 문법에서는 condition의 논리값이 참이면, THEN 절의 result 값을 반환한다.
만약 논리값이 거짓이라면, ELSE 절의 result 값을 반환한다.
이때 ELSE 절이 없으면, NULL을 반환한다.

### 정규 표현식(Regular Expression, RegExr)

> 정규표현식 또는 정규식은 특정한 규칙을 가진 문자열의 집합을 표현하는 데 사용하는 형식 언어이다. 정규 표현식은 많은 텍스트 편집기와 프로그래밍 언어에서 문자열을 검색과 치환을 위해 지원하고 있다.

- 따라서, 데이터베이스의 데이터에 정규 표현식을 사용하여 특정한 규칙을 가진 결과값을 얻을 수 있다.
- LIKE를 사용하여, 일정한 패턴의 결괏값을 얻을 수 있지만, 정규식을 이용하면, 더 다양하고 상세한 결괏값을 얻을 수 있다.

#### 와일드카드(wildcard)

와일드카드(wildcard)란 문자열 내에서 임의의 문자나 문자열을 대체하기 위해 사용되는 기호를 의미한다.
![](https://velog.velcdn.com/images/sanizzang00/post/a00fbb96-da85-4b25-95c3-e65593cfcbde/image.png)

### 정규 표현식 문법

![](https://velog.velcdn.com/images/sanizzang00/post/6c853963-48b5-4614-8703-a9dbaa824344/image.png)

### 날짜와 시간 타입

#### DATE, DATETIME, TIMESTAMP

DATE는 날짜를 저장할 수 있는 타입이다.
기본형식은 'YYYY-MM-DD'이며, 이때 저장할 수 있는 날짜의 범위는 '1000-01-01'부터 '9999-12-31'까지입니다.

DATETIME는 날짜와 함께 시간까지 저장할 수 있는 타입이다.
기본 형식은 'YYYY-MM-DD HH:MM:SS'이며, 이때 저장할 수 있는 범위는 '1000-01-01 00:00:00' 부터 '9999-12-31 23:59:59'까지이다.

TIMESTAMP는 날짜와 시간을 나타내는 타임스탬프를 저장할 수 있는 타입이다.
TIMESTAMP 타입의 필드는 사용자가 별다른 입력을 주지 않으면, 데이터가 마지막으로 입력되거나 변경된 시간이 저장된다.
따라서 데이터의 최종 변경 시각을 저장하고 확인하는 데 유용하게 사용됩니다.
이때 저장할 수 있는 범위는 '1970-01-01 00:00:01' UTC부터 '2038-01-19 03:14:07' UTC까지이다.

#### TIME

TIME은 시간을 저장할 수 있는 타입이다.
기본 형식은 'HH:MM:SS'나 'HHH:MM:SS'이며, 이때 저장할 수 있는 시간의 범위는 '-838:59:59'부터 '838:59:59'까지이다.

#### EXTRACT()

extract() 함수는 날짜/시간 데이터에서 year(년도), month(월), day(일) 과 같은 요소를 추출/검색하는 함수이다.
예를 들면, '2000년 01월 01일 12시 01분 01초' 일때, 현재의 year(2000), month(1), day(1) 의 값을 추출할 수 있다.
![](https://velog.velcdn.com/images/sanizzang00/post/072935e0-a9ca-4374-8935-e6bdb2a3f57e/image.png)
