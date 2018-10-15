# MySQL 8.0 에서의 Descending Index
## Background
### 8.0 GA 중 Desceding Index 가 지원된다는 내용이 있다.
- 명시적으로 Descending Indexes 가 지원된다고 되어있음.
- 8.0 에서는 기존 Backward scan 이 성능이 더 최적화된건지에 대한 여부가 궁금했음.
( - source code 는 확인해보지 않음. )


### asc + desc Key
- ASC + DESC 로 결합된 인덱스를 생성
```
CREATE TABLE t4 (
  tid INT NOT NULL AUTO_INCREMENT,
  TABLE_NAME VARCHAR(64),
  COLUMN_NAME VARCHAR(64),
  ORDINAL_POSITION INT,
  KEY(tid),
  KEY ix_asc_desc (TABLE_NAME asc, COLUMN_NAME desc)
) ENGINE=InnoDB;

INSERT INTO t4 SELECT NULL, TABLE_NAME, COLUMN_NAME, ORDINAL_POSITION FROM information_schema.COLUMNS;
INSERT INTO t4 SELECT NULL, TABLE_NAME, COLUMN_NAME, ORDINAL_POSITION FROM t3; -- // 12번 실행
```

- MySQL 5.7
```
실제 테이블에는 "KEY `ix_asc_desc` (`TABLE_NAME` ASC,`COLUMN_NAME` ASC)" 로 인덱스가 추가 됨.

> explain select * from t2 order by TABLE_NAME asc, COLUMN_NAME desc limit 100;
  => FULL TABLE SCANING
> explain select * from t2 order by TABLE_NAME asc, COLUMN_NAME asc limit 100;
  => INDEX SCANING

```

- MySQL 8.0
```
실제 테이블에 "KEY `ix_asc_desc` (`TABLE_NAME` ASC, `COLUMN_NAME` DESC)" 로 인덱스가 추가 됨.

> explain select * from t2 order by TABLE_NAME asc, COLUMN_NAME desc limit 100;
  => INDEX SCANING
> explain select * from t2 order by TABLE_NAME asc, COLUMN_NAME asc limit 100;
  => FULL TABLE SCANING
```

- 결론
> ASC + DESC 같은 다른 방향을 사용하는 결합 인덱스가 지원된다는 의미.
> 여전히 실제 인덱스에서 ASC+ASC 로 구성된 인덱스에서 ASC+DESC 를 사용할 수는 없음. 



### Backward Scan 성능 측정
```
1. ASC
> SELECT * FROM asc_desc ORDER BY tid ASC  LIMIT 12619775,1;
1 row in set (2.85 sec)
1 row in set (2.84 sec)
1 row in set (2.83 sec)

2. DESC
> SELECT * FROM asc_desc ORDER BY tid DESC LIMIT 12619775,1;
1 row in set (3.59 sec)
1 row in set (3.60 sec)
1 row in set (3.63 sec)
```

- 결론
> Backward Scaning 의 최적화가 이루어진 것은 아닌듯..