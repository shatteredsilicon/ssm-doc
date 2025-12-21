# Advisor - Tuning

This dashboard provides advices for MySQL/MariaDB tuning.

## table_definition_cache

Preconditions
: 
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `info_schema.tables` in **mysqld_exporter.conf**.
: - Set `info_schema.tables.databases = *` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise `table_definition_cache` to be next 2^n of current total table amount (row count in `information_schema.tables` table). And if `(current total table amount) / (next 2^n) > 0.9`, advise the second next 2^n. e.g. current total table amount is **500**, so next 2^n would be **512**, and `500 / 512 ≈ 0.98` is greater than **0.9**. Hence the advice value would be **1024**.
: 
: - min value is `400`
: - max value is `131072`


## table_open_cache

Preconditions
: 
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `global_status` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Round down `table_open_cache` to previous 2^n if `Open_tables` (from output of SHOW GLOBAL STATUS) is smaller than `40% table_open_cache`. Round up to next 2^n if `Open_tables` is greater than `40% table_open_cache` but smaller than `90% table_open_cache`. And Round up to the second next 2^n if it's still greater than 90% of next 2^n.
: 
: - min value is `2000`
: - max value is `262144`


## innodb_log_buffer_size

Preconditions
: 
: - Set `innodb_monitor_enable = all` in MySQL/MariaDB.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `info_schema.innodb_metrics` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system.

Tuning Algorithm
: 
: If the max InnoDB recovery log LSN size since last flush (`Log sequence number` - `Log flushed up to` from output of SHOW ENGINE INNODB STATUS) over past week was spotted at `>= 75% innodb_log_buffer_size`, advise increasing it to the next 2^n of that maximum value, or the second next 2^n if it's greater than `90% of next 2^n`.
: 
: - min value is `262144`
: - max value is `4294967295`


# innodb_redo_log_capacity/innodb_log_file_size

Preconditions
: 
: - Set `innodb_monitor_enable = all` in MySQL/MariaDB.
: - Turn on `global_status` in **mysqld_exporter.conf**.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `info_schema.innodb_metrics` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system since last time the `innodb_redo_log_capacity/innodb_log_file_size` changed.

Tuning Algorithm
: 
: Round down to the previous 2^n if the max checkpoint age over past week is smaller than `40% total redo log size`. Round up to next 2^n if it's greater than `40% total redo log size` but smaller than `90% total redo log size`. And Round up to the second next 2^n if it's still greater than `90% of next 2^n`.
: 
: - min value is `100663296`
: - max value is `137438953472`

## innodb_io_capacity_max

Preconditions
: 
: - For non RDS, turn on `diskstats` in **node_exporter.conf**.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system.

Tuning Algorithm
: 
: Take average disk IO utilization and disk io/s data over the past week, infer what the maximum disk io/s would be (round down to the nearest multiplier of `500`), advise `innodb_io_capacity_max` to be `75% of that maximum value` if it's not already equal to that value.
: 
: - min value is `2000`
: - max value is `18446744073709551615`


## innodb_io_capacity

Preconditions
: 
: - For non RDS, turn on `diskstats` in **node_exporter.conf**.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system.

Tuning Algorithm
: 
: Take average disk IO utilization and disk io/s data over the past week, infer what the maximum disk io/s would be (round down to the nearest multiplier of `500`), advise `innodb_io_capacity` to be `50% of that maximum value` if it's not already equal to that value.
: 
: - min value is `200`
: - max value is `2/3 of innodb_io_capacity_max`


## innodb_read_io_threads

Preconditions
: 
: - For non RDS, turn on `cpu` in **node_exporter.conf**.
: - For non RDS, turn on `diskstats` in **node_exporter.conf**.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `engine_innodb_status` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system since last time the `innodb_read_io_threads` changed.

Tuning Algorithm
: 
: Advise tuning `innodb_read_io_threads` to next 2^n if the maximum pending aio reads (from output of `SHOW ENGINE INNODB STATUS`) over the past week is greater than or equal to `innodb_read_io_threads / 2`, and the disk IO utilization exceeds **75%** at the moment it hits that maximum pending aio reads.
: 
: - min value is `1`
: - max value is `number of CPUs`


## innodb_write_io_threads

Preconditions
: 
: - For non RDS, turn on `cpu` in **node_exporter.conf**.
: - For non RDS, turn on `diskstats` in **node_exporter.conf**.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `engine_innodb_status` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system since last time the `innodb_write_io_threads` changed.

Tuning Algorithm
: 
: Advise tuning `innodb_write_io_threads` to next 2^n if the maximum pending aio writes (from output of `SHOW ENGINE INNODB STATUS`) over the past week is greater than or equal to `innodb_write_io_threads / 2`, and the disk IO utilization exceeds **75%** at the moment it hits that maximum pending aio writes.
: 
: - min value is `1`
: - max value is `number of CPUs`


## thread_cache_size

Preconditions
: 
: - Turn on `global_status` in **mysqld_exporter.conf**.
: - Turn on `global_variables` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise tuning `thread_cache_size` to the amount of unused client threads (`max(threads_connected) - min(threads_connected)` over a rolling 5-min window) if it's less than 80% or more than 110% of current `thread_cache_size` value.
: 
: - min value is `the smaller one of 256 and **max_connections**`
: - max value is `max_connections`


## innodb_buffer_pool_size

Preconditions
: 
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `mysql.innodb_table_stats` or `info_schema.tables` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise tuning `innodb_buffer_pool_size` to **InnoDB Table Index Size** if it's less than 40% or more than 90% of current `innodb_buffer_pool_size` value.
: 
: - min value is `128 MiB`
: - max value is `75% of total RAM`


## key_buffer_size

Preconditions
: 
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `info_schema.tables` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise tuning `innodb_buffer_pool_size` to next MiB of current **MyISAM Table Index Size**.
: 
: - min value is `1 MiB`
: - max value is `75% of total RAM`


## aria_pagecache_buffer_size

Preconditions
: 
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `info_schema.tables` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise tuning `innodb_buffer_pool_size` to next MiB of current **Aria Table Index Size**.
: 
: - min value is `1 MiB`
: - max value is `75% of total RAM`


## binlog_cache_size

Preconditions
: 
: - Turn on `log_bin` in MySQL/MariaDB.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `global_status` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system since last time the `binlog_cache_size` changed.

Tuning Algorithm
: 
: Advise doubling it if the **Hit Ratio** is below 95%.
: 
: - min value is `32768`


## binlog_stmt_cache_size

Preconditions
: 
: - Turn on `log_bin` in MySQL/MariaDB.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `global_status` in **mysqld_exporter.conf**.
: - Have at least 1 week of relevant metric data in the system since last time the `binlog_stmt_cache_size` changed.

Tuning Algorithm
: 
: Advise doubling it if the **Hit Ratio** is below 95%.
: 
: - min value is `32768`


## tmp_table_size

Preconditions
: 
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `global_status` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise tuning `tmp_table_size` to the previous 2^n of maximum `tmp_table_size / **Hit Ratio**` over the past week if the **Hit Ratio** is greater than 0. Otherwise advise rounding it up to the nearest 2^n.  
: 
: - min value is `16 MiB`
: - max value is `1% of total RAM`


## innodb_adaptive_hash_index

Preconditions
: 
: - Set `innodb_monitor_enable = all` in MySQL/MariaDB.
: - Turn on `innodb_adaptive_hash_index` in MySQL/MariaDB.
: - Turn on `global_variables` in **mysqld_exporter.conf**.
: - Turn on `info_schema.innodb_metrics` in **mysqld_exporter.conf**.

Tuning Algorithm
: 
: Advise turning it off if `AHI Hits / ((AHI Hits) + (AHI maintenance (rows added + removed + updated)))` is below 50%.



