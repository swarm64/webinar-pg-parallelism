# WINDOW functions

```sql
EXPLAIN --ANALYZE
SELECT row_number() OVER (PARTITION BY p_brand)
FROM part;
```


## Prepare the data (if you can)

```sql
EXPLAIN --ANALYZE
WITH part_sorted AS (
  SELECT p_brand FROM part ORDER BY p_brand
)
SELECT row_number() OVER (PARTITION BY p_brand)
FROM part_sorted;
```

