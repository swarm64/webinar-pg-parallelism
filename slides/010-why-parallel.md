# Why do you want parallel query execution?

* The short direct answer:
  > It makes your queries go faster.

* The contemporary answer:
  > It improves machine utilization for OLAP/mixed workloads.


# Which queries benefit from parallelism?

* Queries that scan a large amount of data
* Queries with lengthy involved JOIN statements


# Which queries will likely not benefit?

* OLTP-like low-latency queries
* Point-lookup-queries making heavy use of indexes
