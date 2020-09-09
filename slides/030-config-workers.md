# Worker processes, parallel workers, and ... per gather?

* Semantics:
  - `max_worker_processes`: how many _processes_ overall PG is allowed to spawn
  - `max_parallel_workers`: how many _parallel workers_ can be spawned overall
  - `max_parallel_workers_per_gather`: how many _parallel workers_ can be used
    on a portion of the query plan summarized by a "Gather Node".

* Note: a query can have multiple gather nodes


## Example: 24 vCores

```bash
max_worker_processes = 32
max_parallel_workers = 24
max_parallel_workers_per_gather = 12
```

* Run 2 queries at the same time with 12 workers each
* More queries will run with less workers or sequential
* If a query has multiple gather nodes the above might not hold

  > **Rule of thumb**
  >               `max_worker_processes`
  > _larger than_ `max_parallel_workers`
  > _larger than_ `max_parallel_workers_per_gather`


## Recommendation: increase amount of workers

```bash
max_worker_processes = 140
max_parallel_workers = 120
max_parallel_workers_per_gather = 12
```

* 10 queries at max possibly utilizing 12 workers each
* Avoids pitfall of running out of per-gather workers
* Watch out for resource contention/over-utilization


## PostgreSQL limits max parallelism

```bash
max_parallel_workers_per_gather = 24
```

* PostgreSQL limits per-gather parallelism logarithmically
* Limit can be lifted with `swarm64da.maximize_parallel_workers_per_query = on`

