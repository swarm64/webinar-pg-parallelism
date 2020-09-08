


           ____           _                 ____   ___  _
          |  _ \ ___  ___| |_ __ _ _ __ ___/ ___| / _ \| |
          | |_) / _ \/ __| __/ _` | '__/ _ \___ \| | | | |
          |  __/ (_) \__ \ || (_| | | |  __/___) | |_| | |___
          |_|   \___/|___/\__\__, |_|  \___|____/ \__\_\_____|
                             |___/
           ____                 _ _      _ _
          |  _ \ __ _ _ __ __ _| | | ___| (_)___ _ __ ___
          | |_) / _` | '__/ _` | | |/ _ \ | / __| '_ ` _ \ 
          |  __/ (_| | | | (_| | | |  __/ | \__ \ | | | | |
          |_|   \__,_|_|  \__,_|_|_|\___|_|_|___/_| |_| |_|


          Swarm64, Webinar 2020-09-09
          Sebastian Dressler | sebastian@swarm64.com | @theDressler












# Why do you want parallel query execution?

# PostgreSQL Configuration


     ____           _                 ____   ___  _
    |  _ \ ___  ___| |_ __ _ _ __ ___/ ___| / _ \| |
    | |_) / _ \/ __| __/ _` | '__/ _ \___ \| | | | |
    |  __/ (_) \__ \ || (_| | | |  __/___) | |_| | |___
    |_|   \___/|___/\__\__, |_|  \___|____/ \__\_\_____|
                       |___/
      ____             __ _                       _   _
     / ___|___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __
    | |   / _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \ 
    | |__| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | |
     \____\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_|
                            |___/




## Worker processes, parallel workers, and ... per gather?

### Example: 24 vCores

```bash
max_worker_processes = 32
max_parallel_workers = 24
max_parallel_workers_per_gather = 12
```

* Run 2 queries at the same time with 12 workers each
* More queries will run with less workers or sequential
* If a query has multiple gather nodes the above might not hold
* Basically: `max_worker_processes` > `max_parallel_workers` > `max_parallel_workers_per_gather`







### Recommendation: increase amount of workers

```bash
max_worker_processes = 140
max_parallel_workers = 120
max_parallel_workers_per_gather = 12
```

* 10 queries at max possibly utilizing 12 workers each
* Avoids pitfall of running out of per-gather workers
* **Downside:** watch out for resource contention/over-utilization






### PostgreSQL limits max parallelism

```bash
max_parallel_workers_per_gather = 24
```

* PostgreSQL limits per-gather parallelism logarithmically
* Limit can be lifted with `swarm64da.maximize_parallel_workers_per_query = on`






## Cost model

```bash
parallel_tuple_cost = 0.1 # 1.0 default, typically too high
                          # Watch out for CPU operator cost...
parallel_setup_cost = 500
min_parallel_table_scan_size = 0
```








# Are you using Bitmap Heap/Index Scans?

* Low parallelism

```sql
SET enable_indexonlyscan = off;
SET max_parallel_workers_per_gather = 4;
EXPLAIN (ANALYZE) SELECT COUNT(*) FROM customer WHERE c_nationkey = 1;
```


* Medium parallelism

```sql
SET enable_indexonlyscan = off;
SET max_parallel_workers_per_gather = 8;
EXPLAIN (ANALYZE) SELECT COUNT(*) FROM customer WHERE c_nationkey = 1;
```


* High parallelism

```sql
SET enable_indexonlyscan = off;
SET max_parallel_workers_per_gather = 16;
EXPLAIN (ANALYZE) SELECT COUNT(*) FROM customer WHERE c_nationkey = 1;
```


# Parallelism and...

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

-------------------------------------------------------------------------------
                    ____                _____ _
                   |  _ \ _ __ ___     |_   _(_)_ __
                   | |_) | '__/ _ \ _____| | | | '_ \ 
                   |  __/| | | (_) |_____| | | | |_) |
                   |_|   |_|  \___/      |_| |_| .__/
                                               |_|

* JDBC typically connects in "extended query mode", this blocks parallelism
* Use "simple query mode":

```bash
jdbc:postgresql://localhost/test?preferQueryMode=simple
```

-------------------------------------------------------------------------------

## ...WINDOW functions

* Same for `JOIN LATERAL`

## ...INSERT/UPDATE/DELETE

* Is your SQL complex to generate the info you need?

```sql
EXPLAIN
UPDATE customer c
SET c_comment = 'Freight issues'
FROM (
  SELECT c_custkey
  FROM
      customer
    , orders
    , lineitem
    , nation
  WHERE c_custkey = o_custkey
    and l_orderkey = o_orderkey
    and o_orderdate >= date '1993-07-01'
    and o_orderdate < date '1993-07-01' + interval '3' month
    and l_returnflag = 'R'
    and c_nationkey = n_nationkey
  group by c_custkey
) customers
WHERE customers.c_custkey = c.c_custkey;
```


* Make use of temporary tables

```sql
EXPLAIN
CREATE TEMP TABLE customers AS (
  SELECT c_custkey
  FROM
      customer
    , orders
    , lineitem
    , nation
  WHERE c_custkey = o_custkey
    and l_orderkey = o_orderkey
    and o_orderdate >= date '1993-07-01'
    and o_orderdate < date '1993-07-01' + interval '3' month
    and l_returnflag = 'R'
    and c_nationkey = n_nationkey
  group by c_custkey
);

UPDATE customer c
SET c_comment = 'Freight issues'
FROM customers
WHERE customers.c_custkey = c.c_custkey;
```

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
