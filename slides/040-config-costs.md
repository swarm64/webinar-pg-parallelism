# Other configuration settings

## Cost model

* Encourage PostgreSQL to prefer parallel plans by lowering parallel costs
  - `parallel_tuple_cost = 0.1`, default is 1.0 which is typically too high
  - `parallel_setup_cost = 500`, lower the setup cost to encourage earlier use
  - Watch out for `cpu_operator_cost` that it does not become larger than `parallel_tuple_cost`


## Table scan size - the setting you think of least

* Parallel costs are alright but still no parallelism visible?
  - Try `min_parallel_table_scan_size = 0` and then gradually increase
  - Keeping it permanent on "0" can be helpful


## To nested loop or not?

* Quite debateable, but it helps for TPC-H:
  - `enable_nestloop = off`
  - Real-world scenarios likely very different, better keep it enabled
  - The (unexpected) use of nested loops typically hints for a cost-model issue

