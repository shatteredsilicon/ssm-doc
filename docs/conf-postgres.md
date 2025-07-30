# Configuring PostgreSQL for Monitoring

Monitoring PostgreSQL metrics with the [postgres_exporter](https://github.com/wrouesnel/postgres_exporter) is enabled by `ssm-admin add postgresql` command. The `postgresql` alias will set up `postgresql:metrics` and also `linux:metrics` on a host (for more information, see [Adding monitoring services](ssm-admin.md)).

`ssm-admin` supports passing PostgreSQL connection information via following flags:

| Flag         | Description         |
| ------------ | ------------------- |
| `--host`     | PostgreSQL host     |
| `--password` | PostgreSQL password |
| `--port`     | PostgreSQL port     |
| `--user`     | PostgreSQL user     |

An example command line would look like this:

```
ssm-admin add postgresql --host=localhost --password='secret' --port=5432 --user=ssm_user
```

!!! alert alert-info "Note"
    Capturing read and write time statistics is possible only if `track_io_timing` setting is enabled. This can be done either in configuration file or with the following query executed on the running system:

```
ALTER SYSTEM SET track_io_timing=ON;
SELECT pg_reload_conf();
```

## Creating a PostgreSQL User Account to Be Used with SSM

When adding a PostgreSQL instance to monitoring, you can specify the PostgreSQL server superuser account credentials.  However, monitoring with the superuser account is not secure. If you also specify the `--create-user` option, it will create a user with only the necessary privileges for collecting data.

You can also set up the `ssm` user manually with necessary privileges and pass its credentials when adding the instance.

To enable complete PostgreSQL instance monitoring, a command similar to the following is recommended:

```
sudo ssm-admin add postgresql --user root --password root --create-user
```

The superuser credentials are required only to set up the `ssm` user with necessary privileges for collecting data.  If you want to create this user yourself, the following privileges are required:

```
CREATE USER ssm WITH PASSWORD 'pass';
CREATE SCHEMA ssm AUTHORIZATION ssm;
ALTER USER ssm SET SEARCH_PATH TO ssm,public,pg_catalog;
CREATE OR REPLACE VIEW ssm.pg_stat_activity AS SELECT * from pg_catalog.pg_stat_activity;
GRANT SELECT ON ssm.pg_stat_activity TO ssm;
CREATE OR REPLACE VIEW ssm.pg_stat_replication AS SELECT * from pg_catalog.pg_stat_replication;
GRANT SELECT ON ssm.pg_stat_replication TO ssm;
GRANT pg_read_all_settings TO ssm;
GRANT pg_read_all_stats TO ssm;
```

If the `ssm` user already exists, simply pass its credential when you add the instance:

```
sudo ssm-admin add postgresql --user ssm --password pass
```

For more information, run as root `ssm-admin add postgresql --help`.

## Configuring Query Analytics

### Configuring Log file Query Analytics

For Log file Query Analytics to works, you will need to adjust following PostgreSQL settings:

`logging_collector`
: set this to `on`, this setting enables the logging collector, which is a background process that captures log messages sent to stderr and redirects them into log files. See more details about [logging_collector](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOGGING-COLLECTOR).

`log_duration`
: set this to `on`, this setting causes the duration of every completed statement to be logged, so we can get the query time from it. See more details about [log_duration](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DURATION).

`log_statement`
: set this to `none`, this setting controls which SQL statements are logged. But we don't need this as `log_duration = on` also causes the SQL statements to be logged. See more details about [log_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-STATEMENT).

`log_min_duration_statement`
: set this to a proper number, e.g. 1000 (1 second), `0` for logging all statements. This setting causes all SQL statements that run longer than this duration to be logged. See more details about [log_min_duration_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-DURATION-STATEMENT).

`log_destination`
: include `csvlog` or `jsonlog` in this setting, this setting defines the list of desired log destinations. See more details about [log_destination](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DESTINATION).

### Configuring Table Query Analytics

For Table Query Analytics to works, you will need to adjust following PostgreSQL settings:

`shared_preload_libraries`
: include `pg_stat_statements` in this setting, this setting specifies one or more shared libraries to be preloaded at server start.

`pg_stat_statements.track`
: set this to `all` or `top`, this setting controls which statements are counted by the module.

`pg_stat_statements.max`
: set this to a proper number, e.g. 10000, this is the maximum number of statements tracked by the module.

And make sure you have run `CREATE EXTENSION IF NOT EXISTS pg_stat_statements;` for the database used in the Query Analytics system.

!!! alert alert-info "Note"
    For best results, make sure the PostgreSQL user used in the system is a member of the role `pg_read_all_stats`. See also <https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-STATS-VIEWS>.

## Supported versions of PostgreSQL

SSM follows [postgresql.org EOL policy](https://www.postgresql.org/support/versioning/), and thus supports monitoring PostgreSQL version 9.4 and up.  Older versions may work, but will not be supported.

### Setting Up the Required Permissions

We recommends that a PostgreSQL user be configured for `SUPERUSER` level access, in order to gather the maximum amount of data with a minimum amount of complexity. This can be done with the following command for the standalone PostgreSQL installation:

```
CREATE USER ssm_user WITH SUPERUSER ENCRYPTED PASSWORD 'secret';
```

!!! alert alert-info "Note"
    In case of monitoring a PostgreSQL database running on an Amazon RDS instance, the command should look as follows:

    ```
    CREATE USER ssm_user WITH rds_superuser ENCRYPTED PASSWORD 'secret';
    ```
