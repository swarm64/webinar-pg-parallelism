# Other SQL constructs to watch out for

* `WITH` (resp. CTEs):
    - Until PostgreSQL 12 and still sometimes (when used multiple times)
    - Inline or create a `UNLOGGED` table

* `JOIN LATERAL`:
    - Will create a nested loop to process rows
    - Try to find alternatives or pre-filter data as much as you can

* `CREATE TEMP TABLE`:
    - Creation parallel (with `... AS ...`)
    - Usage is not parallel (temporary tables are process local)
