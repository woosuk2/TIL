# ASCENDING INDEX / DESCENDING INDEX
## Background
### 인덱스 내부 구성
- 인덱스 엔트리의 다음 엔트리는 linked list 로 연결되어 구성
- InnoDB 에서 인덱스 엔트리를 4~8개씩 묶어 대표 키를 설정(가장 큰 키 값)
> 이 대표 키를 모은 리스트는 "Page Directory" 





## ASCENDING INDEX / DESCNDING INDEX
### 동작
- ASCENDING 에서는 정순으로 조회를 하기때문에, linked list 를 따라가며 scan
- DESCENDING 의 경우, 하나의 인덱스 엔트리의 대표 키를 선택하여, 마지막 (제일 큰) 값까지 찾아간 후 scan
> 위 동작 때문에, DESCDING INDEX SCAN 의 경우 ASCENDING 보다 느린 성능을 보이게 됨.


```
* TODO TSET
1. ASC
> SELECT * FROM t1 ORDER BY tid ASC LIMIT 12619775,1;
1 row in set (6.33 sec)
1 row in set (6.32 sec)
1 row in set (6.32 sec)


2. DESC
> SELECT * FROM t1 ORDER BY tid DESC LIMIT 12619775,1;
1 row in set (8.32 sec)
1 row in set (8.33 sec)
1 row in set (8.31 sec)
```

