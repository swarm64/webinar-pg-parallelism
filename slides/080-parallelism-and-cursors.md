## ...cursors

* What does it take to get a single row?

```sql
SET enable_indexscan = off;

EXPLAIN ANALYZE
SELECT c_name
FROM customer
ORDER BY c_custkey ASC
LIMIT 1;
```

* A __cursor__ this turns sequential

```sql
SET enable_indexscan = off;

BEGIN;
EXPLAIN ANALYZE
DECLARE c1 CURSOR FOR
  SELECT c_name
  FROM customer
  ORDER BY c_custkey ASC;
-- FETCH c1;
COMMIT;
```

* Rather make use of OFFSET/LIMIT, if you really do not want to fetch all rows at once

