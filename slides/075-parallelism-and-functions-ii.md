# Pitfall: that does not mean function code does not execute parallel...

* Example (**not** marked `PARALLEL SAFE`)

```sql
DROP FUNCTION IF EXISTS customers_in_nation(VARCHAR(25));
CREATE FUNCTION customers_in_nation(in_name VARCHAR(25)) RETURNS BIGINT AS $$
DECLARE
  plan_row TEXT;
BEGIN
  FOR plan_row in
    EXPLAIN ANALYZE
      SELECT COUNT(*)
      FROM customer c
      JOIN nation n ON c.c_nationkey = n.n_nationkey
      WHERE n.n_name = in_name
  LOOP
    RAISE NOTICE '%', plan_row;
  END LOOP;

  RETURN 0;
END
$$ LANGUAGE plpgsql;
```


## Parallelism disabled by config:

```sql
SET max_parallel_workers_per_gather = 0;
SELECT * FROM customers_in_nation('GERMANY');
```


## Parallelism enabled by config:

```sql
SET max_parallel_workers_per_gather = 8;
SELECT * FROM customers_in_nation('GERMANY');
```
