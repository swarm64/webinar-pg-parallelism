# Keep your query parallel on the important parts

* Example: TPC-H query 17

```sql
EXPLAIN
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
```


* Manually de-correlate

```sql
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
  AND l_quantity < tmp.average
```

* Swarm64 DA does the de-correlation for you automatically
* Also: adds "shuffle node" to keep plans parallel to the end
