# The trinity of INSERT, UPDATE & DELETE

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


## Make use of temporary tables

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

