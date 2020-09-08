## ...re-creating tables with INSERT

```sql
TRUNCATE TABLE agg_data;

INSERT INTO agg_data
SELECT ...
FROM raw_data;
```

* Instead: make use of CREATE TABLE AS

```sql
DROP TABLE agg_data;

CREATE TABLE agg_data AS
SELECT ...
FROM raw_data;
```

* Downside: types are auto-deduced and can be not what you expect (e.g. `NUMERIC` vs. `DOUBLE PRECISION`)
* Possibly add explicit casts

-------------------------------------------------------------------------------
     ____                _____ _
    |  _ \ _ __ ___     |_   _(_)_ __
    | |_) | '__/ _ \ _____| | | | '_ \ 
    |  __/| | | (_) |_____| | | | |_) |
    |_|   |_|  \___/      |_| |_| .__/
                                |_|

* Create indexes after you (re-)created the table
* Ensure you have set `max_parallel_maintenance_workers` properly

-------------------------------------------------------------------------------

