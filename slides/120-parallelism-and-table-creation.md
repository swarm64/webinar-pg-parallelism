# Re-creating tables with INSERT

```sql
TRUNCATE TABLE agg_data;

INSERT INTO agg_data
SELECT ...
FROM raw_data;
```


## Instead: make use of CREATE TABLE AS

```sql
DROP TABLE agg_data;

CREATE TABLE agg_data AS
SELECT ...
FROM raw_data;
```


* Be aware of
  - Types are auto-deduced
  - Types can different than exected (e.g. `NUMERIC` vs. `DOUBLE PRECISION`)
  - Consider adding explicit casts

