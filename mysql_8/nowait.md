# MySQL vs MongoDB
## MySQL Document Store
### Document Store
- MySQL 8.0 에서는 Document 를 저장하는 개념이 생겼다. (NoSQL 과 동일)
- RDB 로 풀 수 없는, 비즈니스 로직들을 schemaless 로 풀려는 계획이지 않을까..

### Document 의 형식
- 저장 방식은 MongoDB 와 유사(아니 동일..) 하다.
```
* MySQL 8.0 Shell ( => JS 기반 문법을 지원 )
> db.createCollection("coll1")
<Collection:coll1>

> db.coll1.add({"name":"ws", "age":26})
Query OK, 1 item affected (0.0248 sec)

> db.coll1.find()
[
    {
        "_id": "00005bc098e20000000000000003",
        "age": 26,
        "name": "ws"
    }
]
1 document in set (0.0005 sec)


* 테이블 구조
> show create table coll1;
------------------------------------------------------------------------------------
CREATE TABLE `coll1` (
  `doc` json DEFAULT NULL,
  `_id` varbinary(32) GENERATED ALWAYS AS (json_unquote(json_extract(`doc`,_utf8mb4'$._id'))) STORED NOT NULL,
  PRIMARY KEY (`_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 
------------------------------------------------------------------------------------
1 row in set (0.0021 sec)
```

- Document Store 의 Collection 구조는 MongoDB 의 구조와 동일하게 "_id" 라는 primary key 를 가진 컬럼과, "doc" 라는 document 를 가짐.
- WiredTiger Engine 을 Default Storage Engine 으로 사용하는 MongoDB 의 구조와 전혀 다를게 없음.

db.coll1.createIndex("name").field("name", "INTEGER", false).execute()
db.countryinfo.createIndex("pop").field("demographics.Population", "INTEGER", false).execute()





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

