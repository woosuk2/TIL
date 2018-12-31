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




### Document Store Command
- Collection 생성
```
> db.createCollection("countryinfo");
<Collection:countryinfo>
```

- Document 추가
```
> db.countryinfo.add(
  {
     GNP: .6,
     IndepYear: 1967,
     Name: "Sealand",
     demographics: {
         LifeExpectancy: 79,
         Population: 27
     },
     geography: {
         Continent: "Europe",
         Region: "British Islands",
         SurfaceArea: 193
     },
     government: {
         GovernmentForm: "Monarchy",
         HeadOfState: "Michael Bates"
     }
   })
Query OK, 1 item affected (0.0165 sec)
```

- Document 조회
```
1. find()
> db.countryinfo.find()
[
    {
        "GNP": 0.6,
        "IndepYear": 1967,
        "Name": "Sealand",
        "_id": "00005c10f8630000000000000001",
        "demographics": {
            "LifeExpectancy": 79,
            "Population": 27
        },
        "geography": {
            "Continent": "Europe",
            "Region": "British Islands",
            "SurfaceArea": 193
        },
        "government": {
            "GovernmentForm": "Monarchy",
            "HeadOfState": "Michael Bates"
        }
    }
]

2. 조건 조회
> db.countryinfo.find("GNP=0.6")
[
    {
        "GNP": 0.6,
        "IndepYear": 1967,
        "Name": "Sealand",
        "_id": "00005c10f8630000000000000001",
        "demographics": {
            "LifeExpectancy": 79,
            "Population": 27
        },
        "geography": {
            "Continent": "Europe",
            "Region": "British Islands",
            "SurfaceArea": 193
        },
        "government": {
            "GovernmentForm": "Monarchy",
            "HeadOfState": "Michael Bates"
        }
    }
]
```


- Document Modify
```
1. 필드 UPDATE
> db.countryinfo.modify("GNP=0.6").set("demographics", {LifeExpectancy: 78, Population: 27})
Query OK, 1 item affected (0.0991 sec)

2. 필드 제거
> db.countryinfo.modify("GNP=0.6").unset("GNP")
Query OK, 1 item affected (0.0524 sec)

3. 필드 추가
> db.countryinfo.modify("true").set("Airports", [])
Query OK, 1 item affected (0.0408 sec)

4. 필드에 배열 추가
> db.countryinfo.modify("Name = 'Sealand'").arrayAppend("$.Airports", "ORY")
> db.countryinfo.modify("Name = 'Sealand'").arrayAppend("$.Airports", "ORY3")
> db.countryinfo.find()
[
    {
        "Airports": [
            "ORY",
            "ORY3"
        ],
        "IndepYear": 1967,
        "Name": "Sealand",
        "_id": "00005c10f8630000000000000001",
        "demographics": {
            "LifeExpectancy": 78,
            "Population": 27
        },
        "geography": {
            "Continent": "Europe",
            "Region": "British Islands",
            "SurfaceArea": 193
        },
        "government": {
            "GovernmentForm": "Monarchy",
            "HeadOfState": "Michael Bates"
        }
    }
]

> db.countryinfo.modify("Name = 'Sealand'").arrayAppend("$.Airports[0]", "ORY3")
> db.countryinfo.find()
[
    {
        "Airports": [
            [
                "ORY",
                "ORY3"
            ],
            "ORY2",
            "ORY3"
        ],
        "IndepYear": 1967,
        "Name": "Sealand",
        "_id": "00005c10f8630000000000000001",
        "demographics": {
            "LifeExpectancy": 78,
            "Population": 27
        },
        "geography": {
            "Continent": "Europe",
            "Region": "British Islands",
            "SurfaceArea": 193
        },
        "government": {
            "GovernmentForm": "Monarchy",
            "HeadOfState": "Michael Bates"
        }
    }
]

5. 배열 필드 제거
> db.countryinfo.modify("Name = 'Sealand'").arrayDelete("$.Airports[0][1]")
> db.countryinfo.find()
[
    {
        "Airports": [
            [
                "ORY"
            ],
            "ORY2",
            "ORY3"
        ],
        "IndepYear": 1967,
        "Name": "Sealand",
        "_id": "00005c10f8630000000000000001",
        "demographics": {
            "LifeExpectancy": 78,
            "Population": 27
        },
        "geography": {
            "Continent": "Europe",
            "Region": "British Islands",
            "SurfaceArea": 193
        },
        "government": {
            "GovernmentForm": "Monarchy",
            "HeadOfState": "Michael Bates"
        }
    }
]
```

- Document Remove
```
1. 특정 필드 검색 후 제거
> db.countryinfo.remove("_id = 'SEA'")

2. LIMIT 1
> db.countryinfo.remove("_id = 'SEA'").limit(1)

3. Sort Limit 1
> db.countryinfo.remove("true").sort(["demographics.LifeExpectancy desc"]).limit(1)
※ Sorting 방향을 명시를 하지 않으면 ( ["demographics.LifeExpectancy"] ) ASC 순
```



- Create Index
** 왜인지 모르겠으나 syntax error 남 **
