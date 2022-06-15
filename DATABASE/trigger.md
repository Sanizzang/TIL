### TRIGGER

특정 릴레이션에 변경이 생길 경우 해당 이벤트 전/후에 특정 조건에 따라 조치 수행
(Event-Condition-Action)
명시된 사건(Event)발생할 경우에만 수행(INSERT, UPDATE, DELETE 등)하며 조건을 확인하고 조건 충족 시, 전(BEFORE)이나 후(AFTER)에 조치 수행

```sql
CREATE TRIGGER salary_check
BEFORE INSERT OR UPDATE OF salary, job_id ON employees FOR EACH ROw WHEN(new.job_id<>'AD_VP')
CALL check_sal(:new.job_id,:new.salary, :new.last_name)
```
