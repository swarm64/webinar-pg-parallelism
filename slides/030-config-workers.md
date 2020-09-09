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






