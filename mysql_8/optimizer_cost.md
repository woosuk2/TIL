# Cost Model (Optimizer)
## Background
### 기존 MySQL 5.7 에서는 optimizer 가 통계 정보를 가지고, query plan 을 세운다.
1. query optimizing 에서 필요한 통계정보
- 각 테이블에는 몇개의 row 가 있는지?
- 고유한 value 는 몇가지가 존재하는지?
- 고유한 value 에 대한 분포도가 어떻게 되는지?

2. 만약 통계정보가 없다면, 옵티마이저는 분포도와 상관없이 동일하게 cost 를 산정할 것이다.


### Query Optimizer 의 Data Buffering 고려
- MySQL 8.0 에서는 실행계획을 세울 때, 데이터가 in-memory 인지, on-disk 인지 고려하여 선택.

### Optimizer Histograms
- MySQL 8.0 에서는 histogram statistic 을 사용
- 히스토그램으로 row 에 대한 데이터 분포를 통계 정보로 관리할 수 있음 => 위에서 언급한 2번의 문제를 히스토그램으로 해결할 수 있음
- bucket 을 사용하여 집합을 구성 (DEFAULT : 100개 / 범위 : 1~1024개)
- histogram generate 는 in-memory 로 처리
  => histogram_generation_max_mem_size 만큼 샘플링할 수 있음. (size 내 크기면 full load, size over 시 샘플링)


```
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name] WITH N BUCKETS;
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];


mysql> analyze table t1 update histogram on num1;
+-------+-----------+----------+-------------------------------------------------------------+
| Table | Op        | Msg_type | Msg_text                                                    |
+-------+-----------+----------+-------------------------------------------------------------+
| d1.t1 | histogram | Error    | The column 'num1' is covered by a single-part unique index. |
+-------+-----------+----------+-------------------------------------------------------------+
=> 유니크 인덱스에는 생성되지않음. Primary Key 또한 동일하게 생성되지 않음.(Pkey 도 unique key 이니까)

mysql> analyze table t1 update histogram on t1;
+-------+-----------+----------+-----------------------------------------------+
| Table | Op        | Msg_type | Msg_text                                      |
+-------+-----------+----------+-----------------------------------------------+
| d1.t1 | histogram | status   | Histogram statistics created for column 't1'. |
+-------+-----------+----------+-----------------------------------------------+

* HISTOGRAM 조회 *

mysql> select * from information_schema.COLUMN_STATISTICS \G
*************************** 1. row ***************************
SCHEMA_NAME: d1
 TABLE_NAME: t1
COLUMN_NAME: t1
  HISTOGRAM: {"buckets": [["base64:type254:MQ==", 1.0]], "data-type": "string", "null-values": 0.0, "collation-id": 255, "last-updated": "2018-10-23 11:08:15.524249", "sampling-rate": 1.0, "histogram-type": "singleton", "number-of-buckets-specified": 100}

=> bucket 은 기본적으로 100개가 DEFAULT 로 생성

```

- 히스토그램을 통한 옵티마이저 최적화 예시
```
* EXAMPLE
1. 해당 컬럼의 실제 분포도 = 1.6%
(SELECT COUNT(*) FROM web_page WHERE web_page.wp_char_count BETWEEN 5000 AND 5200) / (SELECT COUNT(*) FROM web_page) AS ratio;
+--------+
| ratio  |
+--------+
| 0.0167 |
+--------+

SELECT CAST(amc AS DECIMAL(15, 4)) / CAST(pmc AS DECIMAL(15, 4)) am_pm_ratio
FROM   (SELECT COUNT(*) amc
        FROM   web_sales,
               household_demographics,
               time_dim,
               web_page
        WHERE  ws_sold_time_sk = time_dim.t_time_sk
               AND ws_ship_hdemo_sk = household_demographics.hd_demo_sk
               AND ws_web_page_sk = web_page.wp_web_page_sk
               AND time_dim.t_hour BETWEEN 9 AND 9 + 1
               AND household_demographics.hd_dep_count = 2
               AND web_page.wp_char_count BETWEEN 5000 AND 5200) at,
       (SELECT COUNT(*) pmc
        FROM   web_sales,
               household_demographics,
               time_dim,
               web_page
        WHERE  ws_sold_time_sk = time_dim.t_time_sk
               AND ws_ship_hdemo_sk = household_demographics.hd_demo_sk
               AND ws_web_page_sk = web_page.wp_web_page_sk
               AND time_dim.t_hour BETWEEN 15 AND 15 + 1
               AND household_demographics.hd_dep_count = 2
               AND web_page.wp_char_count BETWEEN 5000 AND 5200) pt
ORDER  BY am_pm_ratio
LIMIT  100;
실행 시간 : 1 row in set (1.48 sec)

mysql> ANALYZE TABLE web_page UPDATE HISTOGRAM ON wp_char_count WITH 8 BUCKETS;

실행 시간 : 1 row in set (0.50 sec)

=> 히스토그램이 없을 경우, 옵티마이저는 wp_char_count 를 고유한 값의 개수로 분포도를 11% 로 계산하여,결합 순서를 달리함.
   하지만, 두번째의 경우 히스토그램을 통해 1.6% 인 것을 알아차리고 wp_char_count 를 조기에 검사해서 빠르게 처리
```


### 결론
- 기존보다 optimizer 가 똑똑해지긴 했다. 하지만 histogram 의 관리 비용을 생각했을 때, 실 db 에서는 어떻게 사용될지는 감이 오지는 않는듯..