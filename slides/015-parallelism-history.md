# Short history of parallelism in PostgreSQL

* First introduced in PG 9.6, worked with:
  - Sequential scans
  - Hash joins and nested loop joins
  - Two-staged aggregates: partial and final

* PG 10 improves parallelism:
  - Adds bitmap heap scans, index and index only scans
  - Adds merge joins

* PG 11 adds more improvements:
  - Adds parallel append
  - Parallelism much more mature compared to 10 and 9.6
