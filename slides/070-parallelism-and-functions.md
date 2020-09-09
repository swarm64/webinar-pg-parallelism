## ...PL/pgSQL (or other) functions

```sql
DROP FUNCTION IF EXISTS get_nation_id(VARCHAR(25));
CREATE FUNCTION get_nation_id(in_name VARCHAR(25)) RETURNS INT AS $$
DECLARE
  nationkey INT;
BEGIN
  SELECT n_nationkey FROM nation WHERE n_name = in_name INTO nationkey;
  RETURN nationkey;
END
$$ LANGUAGE plpgsql;


EXPLAIN
SELECT COUNT(*)
FROM customer
WHERE c_nationkey = get_nation_id('GERMANY');
```


* Let PostgreSQL know whether your function is parallel safe.

```sql
DROP FUNCTION IF EXISTS get_nation_id(VARCHAR(25));
CREATE FUNCTION get_nation_id(in_name VARCHAR(25)) RETURNS INT AS $$
DECLARE
  nationkey INT;
BEGIN
  SELECT n_nationkey FROM nation WHERE n_name = in_name INTO nationkey;
  RETURN nationkey;
END
$$ LANGUAGE plpgsql PARALLEL SAFE;
--                  ^^^^^^^^^^^^^

EXPLAIN
SELECT COUNT(*)
FROM customer
WHERE c_nationkey = get_nation_id('GERMANY');
```


* Pitfall: that does not mean function code does not execute parallel...
* For example (**not** marked `PARALLEL SAFE`):

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


* Parallelism disabled by config:

```sql
SET max_parallel_workers_per_gather = 0;
SELECT * FROM customers_in_nation('GERMANY');
```


* Parallelism enabled by config:

```sql
SET max_parallel_workers_per_gather = 8;
SELECT * FROM customers_in_nation('GERMANY');
```
