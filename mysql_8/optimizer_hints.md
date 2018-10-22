# Optimizer Hints
## Background
### 기존 5.7 version 까지 optimizer hint 는 몇개 없음.
- 8.0 에서는 쿼리 튜닝 레벨에서 optimizing 이 hint 로 가능할까?


### INDEX_MERGE / NO_INDEX_MERGE
- 옵티마이저 스위치를 변경하지 않고, 각 쿼리에 대한 index_merge 동작을 제어할 수 있다. (....?)
- 좀 더 테스트가 필요할 듯..

### JOIN_FIXED_ORDER, JOIN_ORDER, JOIN_PREFIX
- 조인 실행 순서를 컨트롤할 수 있음. (RULE 을 정할 수 있을듯?)

### SET_VAR
- 쿼리마다, system variables 값을 변경하여 실행 가능하다.
```
* step. 1 
SET @old_var1 = @var1

* step. 2
SET var1 = setting_value

* step. 3
SELECT * FROM t1 (쿼리 실행)

* step. 4
SET @var1 = @old_var1 (원복)

* Example
SELECT
/*+ SET_VAR(optimizer_switch = 'mrr_cost_based=off')
    SET_VAR(max_heap_table_size = 1G) */ *
FROM t1;

* 지원 가능한 변수
auto_increment_increment
auto_increment_offset
big_tables
bulk_insert_buffer_size
default_tmp_storage_engine
div_precision_increment
end_markers_in_json
eq_range_index_dive_limit
foreign_key_checks
group_concat_max_len
insert_id
internal_tmp_mem_storage_engine
join_buffer_size
lock_wait_timeout
max_error_count
max_execution_time
max_heap_table_size
max_join_size
max_length_for_sort_data
max_points_in_geometry
max_seeks_for_key
max_sort_length
optimizer_prune_level
optimizer_search_depth variables
optimizer_switch
range_alloc_block_size
range_optimizer_max_mem_size
read_buffer_size
read_rnd_buffer_size
sort_buffer_size
sql_auto_is_null
sql_big_selects
sql_buffer_result
sql_mode
sql_safe_updates
sql_select_limit
timestamp
tmp_table_size
updatable_views_with_limit
unique_checks
windowing_use_high_precision
```
- 잘 사용하면 좋은 튜닝 포인트가 될 수 있을듯.



- 결론
> 기존 use/force index 만 주로 사용하던 hint 에서, 잘 사용하면 유용한 힌트가 많이 생긴 것 같다.