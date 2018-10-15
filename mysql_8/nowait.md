# NOWAIT / SKIP LOCKED
## Background
### 기존 상황
- ROW 가 UPDATE 또는 SELECT ... FOR UPDATE 로 잠겨있으면, 다른 트랜잭션은 액세스를 위해 wait
> 일부 케이스에선, WAIT 가 아니라 FAIL 이라도 즉각적인 반환이 필요할 수 있음.






## NOWAIT 
### 동작
- LOCK 을 유발할 쿼리에 NOWAIT 을 함께 사용
- WAIT 을 해야할 트랜잭션은 즉시 처리. (대신 error return)


```
* TODO TEST LIST
1. LOCK WAIT 상황 재현
session 1) 
> start transaction;
> select * from t1 where id=1 for update;

session 2)
> select * from t1 where id=1 for update; << LOCKING


2. NOWAIT 사용
session 1)
> start transaction;
session 1) 
> start transaction;
> select * from t1 where id=1 for update;

session 2)
> select * from t1 where id=1 for update;
> select * from t1 where id=1 for update skip locked;
  ==> Empty Set
```

