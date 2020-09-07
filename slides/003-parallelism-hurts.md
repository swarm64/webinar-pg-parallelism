
# Parallelism Hurts

## Bitmap Index Scan

```sql
CREATE TABLE sampletable (x numeric);

INSERT INTO sampletable
SELECT random() * 10000
FROM generate_series(1, 10000000);

CREATE INDEX idx_x ON sampletable(x);

VACUUM sampletable;
ANALYZE sampletable;
```

```sql
explain SELECT * FROM sampletable WHERE x < 423;
```
