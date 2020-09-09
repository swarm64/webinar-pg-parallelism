# Cursors

## What does it take to get a single row?

```sql
SET enable_indexscan = off;

EXPLAIN ANALYZE
SELECT c_name
FROM customer
ORDER BY c_custkey ASC
LIMIT 1;
```


## This turns sequential when using a _cursor_

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


## Suggestions

- Fetch all rows at once
- Make use of OFFSET/LIMIT

