vim: set ft=md

# PostgreSQL Configuration

## The case of parallel workers

**Default config**:

```bash
max_worker_processes = 8
max_parallel_workers = 8
max_parallel_workers_per_gather = 2
```


## But my machine has 24 vCores?

```bash
max_worker_processes = 32
max_parallel_workers = 24
max_parallel_workers_per_gather = 12
```

-> Enables to run 2 queries at the same time, both can use 12 parallel workers
-> 3rd query will run but sequential
-> Keep in mind: `max_worker_processes`


## Is it safe to?

```bash
max_worker_processes = 140
max_parallel_workers = 120
max_parallel_workers_per_gather = 12
```

-> Yes, but watch out for resource contention/over-utilization


## But I want to use all 24 vCores!

```bash
max_parallel_workers_per_gather = 24
```

-> PostgreSQL limits parallelism logarithmically
-> Limit can be lifted with `swarm64da.maximize_parallel_workers_per_query = on`
