# Keep your query parallel on the important parts

## Example: TPC-H query 17

```sql
EXPLAIN --ANALYZE
SELECT SUM(l_extendedprice) / 7.0 AS avg_yearly
FROM
    lineitem
  , part
WHERE p_partkey = l_partkey
  AND p_brand = 'Brand#24'
  AND p_container = 'WRAP JAR'
  AND l_quantity < (
    SELECT 0.2 * AVG(l_quantity)
    FROM lineitem
    WHERE l_partkey = p_partkey
    --    ^^^^^^^^^^^^^^^^^^^^^
    --      THIS BAD BOY HERE
  );

-- Runtime: 150s
```


## Manually de-correlate

```sql
EXPLAIN --ANALYZE
SELECT SUM(l_extendedprice) / 7.0 AS avg_yearly
FROM
    lineitem
  , part
  , (
      SELECT
          0.2 * AVG(l_quantity) AS average
        , l_partkey AS correlated_key
      FROM
          lineitem l
        , part p
      WHERE l.l_partkey = p.p_partkey
        AND p_brand = 'Brand#24'
        AND p_container = 'WRAP JAR'
      GROUP BY l_partkey
    ) tmp
WHERE p_partkey = l_partkey
  AND p_brand = 'Brand#24'
  AND p_container = 'WRAP JAR'
  AND p_partkey = tmp.correlated_key
  AND l_quantity < tmp.average;

-- Runtime: 19s
```


## Swarm64 DA does the de-correlation for you automatically

* Adds additional nodes to keep plan parallel
* Improved more-efficient join
* Columnstore index for fast data access

```sql
SET swarm64da.enable_columnstore = on;
SET swarm64da.enable_join = on;
SET swarm64da.enable_not_in_to_not_exists = on;
SET swarm64da.enable_seqscan = on;
SET swarm64da.enable_shuffle = on;
SET swarm64da.enable_shuffle_clause_minimization = on;
SET swarm64da.enable_shuffled_aggregate = on;
SET swarm64da.enable_shuffled_distinct = on;
SET swarm64da.enable_unnesting = on;
SET swarm64da.maximize_parallel_workers_per_query = on;

EXPLAIN ANALYZE
SELECT SUM(l_extendedprice) / 7.0 AS avg_yearly
FROM
    lineitem
  , part
WHERE p_partkey = l_partkey
  AND p_brand = 'Brand#24'
  AND p_container = 'WRAP JAR'
  AND l_quantity < (
    SELECT 0.2 * AVG(l_quantity)
    FROM lineitem
    WHERE l_partkey = p_partkey
  );

-- Runtime: 24s
```
