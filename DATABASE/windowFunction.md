### 윈도우 함수(Window Function)란?

- 윈도우 함수는 행과 행간의 관계를 정의하기 위해 제공되는 함수이다.
- 순위, 합계, 평균, 행 위치 등을 조작할 수 있다.

### WINDOW FUNCTION 종류

WINDOW FUNCTION은 크게 5가지 그룹으로 분류할 수 있다.

1. 그룹 내 순위(RANK) 관련 함수: RANK, DENSE_RANK, ROW_NUMBER
2. 그룹 내 집계(AGGREGATE) 관련 함수: SUM, MAX, MIN, AVG, COUNT (sql server는 OVER 절의 ORDER BY 지원 X)
3. 그룹 내 행 순서 관련 함수: FIRST_VALUE, LAST_VALUE, LAG, LEAD (오라클에서만 지원)
4. 그룹 내 비율 관련 함수: CUME_DIST, PERCENT_RANK, NTILE, RATIO_TO_REPORT
5. 선형 분석을 포함한 통계 분석 함

### 윈도우 함수의 구조

```sql
SELECT WINDOW_FUNCTION(ARGUMENTS)
    OVER (PARTITION BY 컬럼 ORDER BY WINDOWING 절)
FROM 테이블명;
```

- WINDOW_FUNCTION: 윈도우 함수
- ARGUMENTS(인수) - 윈도우 함수에 따라 0~N개의 인수를 설정한다.
- PARTITION BY - 전체 집합을 기준에 의해 소그룹으로 나눈다.
- ORDER BY - 항목에 의해 정렬한다.
- WINDOWING - 행 기준의 범위를 정한다.
  ROWS는 물리적 결과의 행 수, RANGE는 논리적인 값에 의한 범위이다.
