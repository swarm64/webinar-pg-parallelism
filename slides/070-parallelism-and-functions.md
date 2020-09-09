# PL/pgSQL (or other) functions

* PostgreSQL plays it safe
  > Not parallel safe means no parallelism.


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


## Let PostgreSQL know whether your function is parallel safe.

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

