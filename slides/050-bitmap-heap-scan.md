# Bitmap heap/index scans

## Low parallelism

```sql
SET enable_indexonlyscan = off;
SET max_parallel_workers_per_gather = 4;
EXPLAIN (ANALYZE) SELECT COUNT(*) FROM customer WHERE c_nationkey = 1;
```


## Medium parallelism

```sql
SET enable_indexonlyscan = off;
SET max_parallel_workers_per_gather = 8;
EXPLAIN (ANALYZE) SELECT COUNT(*) FROM customer WHERE c_nationkey = 1;
```


## High parallelism

```sql
SET enable_indexonlyscan = off;
SET max_parallel_workers_per_gather = 16;
EXPLAIN (ANALYZE) SELECT COUNT(*) FROM customer WHERE c_nationkey = 1;
```

