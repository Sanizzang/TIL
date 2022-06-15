### Index란?

(검색을 위해) 임의의 규칙대로 부여된, 임의의 대상을 가리키는 무언가.

### Index in Database

데이터베이스는 내가 원하는 데이터를 어떻게 찾아오는 걸까?
왜 데이터가 많이질수록 점점 느려질까?
왜 조인만 수행하면 느릴까?
왜 쿼리가 느릴까?

### Clustered vs Non-Clustered

Cluster: 군집
Clustered: 군집화
Clustered Index: 군집화된 인덱스
인덱스와 데이터가 군집

```sql
CREATE INDEX 인덱스이름
ON 테이블이름 (필드이름1, 필드이름2, ...)
```
