# Managing SSM Client

Use the `ssm-admin` tool to manage SSM Client.

In this chapter

[TOC]

## USAGE

```
ssm-admin [OPTIONS] [COMMAND]
```

!!! alert alert-info "Note"
    The **ssm-admin** tool requires root access (you should either be logged in as a user with root privileges or be able to run commands with **sudo**).

To view all available commands and options, run **ssm-admin** without any commands or options:

```
$ sudo ssm-admin
```

## OPTIONS

The following options can be used with any command:

`-c`, `--config-file`
:    Specify the location of SSM configuration file (default `/opt/ss/ssm-client/ssm.yml`).

`-h`, `--help`
:     Print help for any command and exit.

`-v`, `--version`
:     Print version of SSM Client.

`--verbose`
:     Print verbose output.

## COMMANDS

[**ssm-admin add**](#adding-monitoring-services)
:     Add a monitoring service.

[Adding annotations](#adding-annotations)
:     Add an annotation

[**ssm-admin check-network**](#checking-network-connectivity)
:     Check network connection between SSM Client and SSM Server.

[**ssm-admin config**](#configuring-ssm-client)
:     Configure how SSM Client communicates with SSM Server.

[**ssm-admin help**](#getting-help-for-any-command)
:     Print help for any command and exit.

[**ssm-admin info**](#getting-information-about-ssm-client)
:     Print information about SSM Client.

[**ssm-admin list**](#listing-monitoring-services)
:     List all monitoring services added for this SSM Client.

[**ssm-admin ping**](#pinging-ssm-server)
:     Check if SSM Server is alive.

[**ssm-admin purge**](#purging-metrics-data)
:     Purge metrics data on SSM Server.

[**ssm-admin remove**, **ssm-admin rm**](#removing-monitoring-services)
:     Remove monitoring services.

[**ssm-admin repair**](#removing-orphaned-services)
:     Remove orphaned services.

[**ssm-admin restart**](#restarting-monitoring-services)
:     Restart monitoring services.

[**ssm-admin show-passwords**](#getting-passwords-used-by-ssm-client)
:     Print passwords used by SSM Client (stored in the configuration file).

[**ssm-admin start**](#starting-monitoring-services)
:     Start monitoring service.

[**ssm-admin stop**](#stopping-monitoring-services)
:     Stop monitoring service.

[**ssm-admin uninstall**](#cleaning-up-before-uninstall)
:     Clean up SSM Client before uninstalling it.

## Adding monitoring services

Use the **ssm-admin add** command to add monitoring services.

### USAGE

```
$ ssm-admin add [OPTIONS] [SERVICE]
```

When you add a monitoring service **ssm-admin** automatically creates and sets up a service in the operating system. You can tweak the **systemd** configuration file and change its behavior.

For example, you may need to disable the HTTPS protocol for the Prometheus exporter associated with the given service. To accomplish this task, you need to remove all SSL related options.

Run the following commands as root or by using the **sudo** command:

1. Open the **systemd** unit file associated with the monitoring service that you need to change, such as `ssm-mysql-metrics-42002.service`.

    ```
    $ cd /etc/systemd/system
    $ cat ssm-mysql-metrics-42002.service
    ```

2. Remove the SSL related configuration options (key, cert) from the **systemd** unit file or init.d startup script. Examples of the systemd Unit File highlights the SSL related options in the **systemd** unit file.

    The following code demonstrates how you can remove the options using the **sed** command. (If you need more information about how **sed** works, see the documentation of your system).

    ```
    $ sed -e -i.backup 's/-web.ssl[^ ]\+[^->]*//g' ssm-mysql-metrics-42002.service
    ```

3. Reload **systemd**:

    ```
    $ systemctl daemon-reload
    ```

4. Restart the monitoring service by using **ssm-admin restart**:

    ```
    $ ssm-admin restart mysql:metrics
    ```

### OPTIONS

The following option can be used with the **ssm-admin add** command:

`--dev-enable`
:     Enable experimental features.

`--disable-ssl`
:     Disable (otherwise enabled) SSL for the connection between SSM Client and SSM Server. Turning off SSL encryption for the data acquired from some objects of monitoring allows to decrease the overhead for a SSM Server connected with a lot of nodes.

`--service-port`
:  Specify the service port.

You can also use global options that apply to any other command.

### SERVICES

Specify a monitoring service alias, along with any relevant additional arguments.

For more information, run **ssm-admin add** `--help`.

### Adding external monitoring services

The **ssm-admin add** command is also used to add external monitoring services. This command adds an external monitoring service assuming that the underlying Prometheus exporter is already set up and accessible. The default scrape timeout is 10 seconds, and the interval equals to 1 minute.

To add an external monitoring service use the `external:service` monitoring service followed by the port number, name of a Prometheus job. These options are required. To specify the port number the `--service-port` option.

```
$ ssm-admin add external:service --service-port=9187 postgresql

ssm-admin 1.12.0

SSM Server      | 127.0.0.1:80
Client Name     | ssm
Client Address  | 172.17.0.1
Service Manager | linux-systemd

...

Job name    Scrape interval  Scrape timeout  Metrics path  Scheme  Target           Labels              Health
postgresql  10s              1m              /metrics      http    172.17.0.1:9187  instance="ssm"
```

By default, the **ssm-admin add** command automatically creates the name of the host to be displayed in the Host field of the *Advanced Data Exploration* dashboard where the metrics of the newly added external monitoring service will be displayed. This name matches the name of the host where **ssm-admin** is installed. You may choose another display name when adding the `external:service` monitoring service giving it explicitly after the Prometheus exporter name.

You may also use the `external:metrics` monitoring service. When using this option, you refer to the exporter by using a URL and a port number. The following example adds an external monitoring service which monitors a PostgreSQL instance at 192.168.200.1, port 9187. After the command completes, the **ssm-admin list** command shows the newly added external exporter at the bottom of the command’s output:

Run this command as root or by using the **sudo** command

```
$ ssm-admin add external:metrics postgresql 192.168.200.1:9187

SSM Server      | 192.168.100.1
Client Name     | ssm
Client Address  | 192.168.200.1
Service Manager | linux-systemd

-------------- -------- ----------- -------- ------------ --------
SERVICE TYPE   NAME     LOCAL PORT  RUNNING  DATA SOURCE  OPTIONS
-------------- -------- ----------- -------- ------------ --------
linux:metrics  ssm      42000       YES                 -


Name      Scrape interval  Scrape timeout  Metrics path  Scheme  Instances
postgres  10s              1m              /metrics      http    192.168.200.1:9187
```

### Passing options to the exporter

**ssm-admin add** sends all options which follow `--` (two consecutive dashes delimited by whitespace) to the Prometheus exporter that the given monitoring services uses. Each exporter has its own set of options.

Run the following commands as root or by using the **sudo** command.

Passing `--collect.perf_schema.eventsstatements` to the `mysql:metrics` monitoring service:

```
$ ssm-admin add mysql:metrics -- --collect.perf_schema.eventsstatements
```

Passing `--collect.perf_schema.eventswaits=false` to the `mysql:metrics` monitoring service:

```
$ ssm-admin add mysql:metrics -- --collect.perf_schema.eventswaits=false
```

The section [Exporters Overview](index.exporter-option.md) contains all option grouped by exporters.

### Passing SSL parameters to the mongodb monitoring service

SSL/TLS related parameters are passed to an SSL enabled MongoDB server as monitoring service parameters along with the **ssm-admin add** command when adding the `mongodb:metrics` monitoring service.

Run this command as root or by using the **sudo** command

```
$ ssm-admin add mongodb:metrics -- --mongodb.tls
```

### Supported SSL/TLS Parameters

| Parameter                                   | Description                               |
| ------------------------------------------- | ------------------------------------------|
| `--mongodb.tls`                             | Enable a TLS connection with mongo server |
| `--mongodb.tls-ca`  *string*                | A path to a PEM file that contains the CAs that are trusted for server connections. *If provided*: MongoDB servers connecting to should present a certificate signed by one of these CAs. *If not provided*: System default CAs are used. |
| `--mongodb.tls-cert` *string*               | A path to a PEM file that contains the certificate and, optionally, the private key in the PEM format. This should include the whole certificate chain. *If provided*: The connection will be opened via TLS to the MongoDB server. |
| `--mongodb.tls-disable-hostname-validation` | Do hostname validation for the server connection. |
| `--mongodb.tls-private-key` *string*        | A path to a PEM file that contains the private key (if not contained in the `mongodb.tls-cert` file). |

!!! alert alert-info "Note"
    SSM does not support passing SSL/TLS related parameters to `mongodb:queries`.

```
$ mongod --dbpath=DATABASEDIR --profile 2 --slowms 200 --rateLimit 100
```

## Adding general system metrics service

Use the `linux:metrics` alias to enable general system metrics monitoring.

### USAGE

```
$ ssm-admin add linux:metrics [NAME] [OPTIONS]
```

This creates the `ssm-linux-metrics-42000` service that collects local system metrics for this particular OS instance.

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following option can be used with the `linux:metrics` alias:

`--force`
: Force to add another general system metrics service with a different name for testing purposes.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

For more information, run **ssm-admin add** `linux:metrics --help`.

## Extending metrics with textfile collector

**Versionadded:** New in version 1.16.0.

While SSM provides an excellent solution for system monitoring, sometimes you may have the need for a metric that’s not present in the list of `node_exporter` metrics out of the box. There is a simple method to extend the list of available metrics without modifying the `node_exporter` code. It is based on the textfile collector.

Starting from version 1.16.0, this collector is enabled for the `linux:metrics` in SSM Client by default.

The default directory for reading text files with the metrics is `/opt/ss/ssm-client/textfile-collector`, and the exporter reads files from it with the `.prom` extension. By default it contains an example file  `example.prom` which has commented contents and can be used as a template.

You are responsible for running a cronjob or other regular process to generate the metric series data and write it to this directory.

### Example - collecting docker container information

This example will show you how to collect the number of running and stopped docker containers on a host. It uses a `crontab` task, set with the following lines in the cron configuration file (e.g. in `/etc/crontab`):

```
*/1 * * * *     root   echo -n "" > /tmp/docker_all.prom; /usr/bin/docker ps -a | sed -n '1!p'| /usr/bin/wc -l | sed -ne 's/^/node_docker_containers_total /p' >> /opt/ss/ssm-client/docker_all.prom;
*/1 * * * *     root   echo -n "" > /tmp/docker_running.prom; /usr/bin/docker ps | sed -n '1!p'| /usr/bin/wc -l | sed -ne 's/^/node_docker_containers_running_total /p' >>/opt/ss/ssm-client/docker_running.prom;
```

The result of the commands is placed into the `docker_all.prom` and `docker_running.prom` files and read by exporter.

The first command executed by cron is rather simple: the destination text file is cleared by executing `echo -n ""`, then a list of running and closed containers is generated with `docker ps -a`, and finally `sed` and `wc` tools are used to count the number of containers in this list and to form the output file which looks like follows:

```
node_docker_containers_total 2
```

The second command is similar, but it counts only running containers.

## Adding MySQL query analytics service

Use the `mysql:queries` alias to enable MySQL query analytics.

### USAGE

```
ssm-admin add mysql:queries [NAME] [OPTIONS]
```

This creates the `ssm-mysql-queries-0` service that is able to collect QAN data for multiple remote MySQL server instances.

The **ssm-admin add** command is able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

!!! alert alert-warning "Important"
    If you connect MySQL Server version 8.0, make sure it is started with the `default_authentication_plugin` set to the value **mysql_native_password**.

    You may alter your SSM user and pass the authentication plugin as a parameter:

    ```
    mysql> ALTER USER ssm@'localhost' IDENTIFIED WITH mysql_native_password BY '$eCR8Tp@s$w*rD';
    ```

### OPTIONS

The following options can be used with the `mysql:queries` alias:

`--create-user`
: Create a dedicated MySQL user for SSM Client (named `ssm`).

`--create-user-maxconn`
: Specify maximum connections for the dedicated MySQL user (default is 10).

`--create-user-password`
: Specify password for the dedicated MySQL user.

`--defaults-file`
: Specify path to `my.cnf`.

`--disable-queryexamples`
: Disable collection of query examples.

`--slow-log-rotation`
: Do not manage *slow log* files by using SSM. Set this option to *false* if you intend to manage *slow log* files by using a third party tool.  The default value is *true*

`--force`
:     Force to create or update the dedicated MySQL user.

`--host`
:     Specify the MySQL host name.

`--password`
:     Specify the password for MySQL user with admin privileges.

`--port`
:     Specify the MySQL instance port.

`--query-source`
:     Specify the source of data:

    * `auto`: Select automatically (default).
    * `slowlog`: Use the slow query log.
    * `perfschema`: Use Performance Schema.

`--retain-slow-logs`
:    Specify the maximum number of files of the *slow log* to keep automatically. The default value is 1 file.

`--socket`
: Specify the MySQL instance socket file.

`--user`
: Specify the name of MySQL user with admin privileges.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

### DETAILED DESCRIPTION

When adding the MySQL query analytics service, the **ssm-admin** tool will attempt to automatically detect the local MySQL instance and MySQL superuser credentials.  You can use options to provide this information, if it cannot be detected automatically.

You can also specify the `--create-user` option to create a dedicated `ssm` user on the MySQL instance that you want to monitor. This user will be given all the necessary privileges for monitoring, and is recommended over using the MySQL superuser.

For example, to set up remote monitoring of QAN data on a MySQL server located at 192.168.200.2, use a command similar to the following:

```
$ ssm-admin add mysql:queries --user root --password root --host 192.168.200.2 --create-user
```

QAN can use either the *slow query log* or *Performance Schema* as the source. By default, it chooses the *slow query log* for a local MySQL instance and *Performance Schema* otherwise. For more information about the differences, see [Configuring Performance Schema](conf-mysql.md#configuring-performance-schema).

You can explicitly set the query source when adding a QAN instance using the `--query-source` option.

For more information, run **ssm-admin add** `mysql:queries --help`.

## Adding MySQL metrics service

Use the `mysql:metrics` alias to enable MySQL metrics monitoring.

### USAGE

```
$ ssm-admin add mysql:metrics [NAME] [OPTIONS]
```

This creates the `ssm-mysql-metrics-42002` service that collects MySQL instance metrics.

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following options can be used with the `mysql:metrics` alias:

`--create-user`
: Create a dedicated MySQL user for SSM Client (named `ssm`).

`--create-user-maxconn`
: Specify maximum connections for the dedicated MySQL user (default is 10).

`--create-user-password`
: Specify password for the dedicated MySQL user.

`--defaults-file`
: Specify the path to `my.cnf`.

`--disable-binlogstats`
: Disable collection of binary log statistics.

`--disable-processlist`
: Disable collection of process state metrics.

`--disable-tablestats`
: Disable collection of table statistics.

`--disable-tablestats-limit`
: Specify the maximum number of tables for which collection of table statistics is enabled (by default, the limit is 1 000 tables).

Disabling the table stats will disable the following collectors in `mysqld_exporter.conf`:
```
info_schema.tables
auto_increment.columns
perf_schema.tableiowaits
perf_schema.indexiowaits
perf_schema.tablelocks
info_schema.tablestats
```

If you are getting telemetry dropouts due to timeouts caused by gathering detailed telemetry from many thousands of tables, change these to 0 in your `mysqld_exporter.conf` and restart the mysql:metric collector by running
```
ssm-admin restart mysql:metrics
```

`--disable-userstats`
: Disable collection of user statistics.

`--force`
: Force to create or update the dedicated MySQL user.

`--host`
: Specify the MySQL host name.

`--password`
: Specify the password for MySQL user with admin privileges.

`--port`
: Specify the MySQL instance port.

`--socket`
: Specify the MySQL instance socket file.

`--user`
: Specify the name of MySQL user with admin privileges.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

### DETAILED DESCRIPTION

When adding the MySQL metrics monitoring service, the **ssm-admin** tool attempts to automatically detect the local MySQL instance and MySQL superuser credentials.  You can use options to provide this information, if it cannot be detected automatically.

You can also specify the `--create-user` option to create a dedicated `ssm` user on the MySQL host that you want to monitor.  This user will be given all the necessary privileges for monitoring, and is recommended over using the MySQL superuser.

For example, to set up remote monitoring of MySQL metrics on a server located at 192.168.200.3, use a command similar to the following:

```
$ ssm-admin add mysql:metrics --user root --password root --host 192.168.200.3 --create-user
```

For more information, run **ssm-admin add** `mysql:metrics` `--help`.

## Adding query filtering

To set the filter for MySQL instances that are on the client side (with the **ssm-admin** tool), you can pass the `--qan-filter-omit` argument when you run `ssm-admin add mysql ...`, like this:

```
ssm-admin add mysql ... --qan-filter-omit COMMIT,RESET,PING,PREPARE,ROLLBACK,SET
```

After you have added the instance, you can also go to the SSM Query Analytics Settings page to adjust the filter.

The default value of the filter is empty, case insensitive, and it should be comma separated when you set it up - for all `QAN_FILTER_OMIT` environment variables, `--qan-filter-omit` argument and the filter option on the SSM Query Analytics Settings page. 

## Adding MongoDB query analytics service

Use the `mongodb:queries` alias to enable MongoDB query analytics.

### USAGE

```
ssm-admin add mongodb:queries [NAME] [OPTIONS]
```

This creates the `ssm-mongodb-queries-0` service that is able to collect QAN data for multiple remote MongoDB server instances.

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following options can be used with the `mongodb:queries` alias:

`--uri`
: Specify the MongoDB instance URI with the following format:

    ```
    [mongodb://][user:pass@]host[:port][/database][?options]
    ```

    By default, it is `localhost:27017`.

    !!! alert alert-warning "Important"

        In cases when the password contains special symbols like the *at* (@) symbol, the host might not not be detected correctly. Make sure that you insert the password with special characters replaced with their escape sequences. The simplest way is to use the `encodeURIComponent` JavaScript function.

        For this, open the web console of your browser (usually found under *Development tools*) and evaluate the following expression, passing the password that you intend to use:

        ```
	    encodeURIComponent('$ecRet_pas$w@rd')
        "%24ecRet_pas%24w%40rd"
        ```

        !!! seealso "Related Information"
        	MDN Web Docs: encodeURIComponent
	        <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent>

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

!!! alert alert-info "Note"
    SSM does not support passing SSL/TLS related parameters to `mongodb:queries`.

For more information, run **ssm-admin add** `mongodb:queries` `--help`.

## Adding MongoDB metrics service

Use the `mongodb:metrics` alias to enable MongoDB metrics monitoring.

### USAGE

```
$ ssm-admin add mongodb:metrics [NAME] [OPTIONS]
```

This creates the `ssm-mongodb-metrics-42003` service that collects local MongoDB metrics for this particular MongoDB instance.

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following options can be used with the `mongodb:metrics` alias:

`--cluster`
: Specify the MongoDB cluster name.

`--uri`
: Specify the MongoDB instance URI with the following format:

    ```
    [mongodb://][user:pass@]host[:port][/database][?options]
    ```

    By default, it is `localhost:27017`.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

For more information, run **ssm-admin add** `mongodb:metrics` `--help`.

### Monitoring a cluster

When using SSM to monitor a cluster, you should enable monitoring for each instance by using the **ssm-admin add** command. This includes each member of replica sets in shards, mongos, and all configuration servers. Make sure that for each instance you supply the cluster name via the `--cluster` option and provide its URI via the `--uri` option.

Run this command as root or by using the **sudo** command. This examples uses *127.0.0.1* as a URL.

```
$ ssm-admin add mongodb:metrics \
--uri mongodb://127.0.0.1:<port>/admin <instance name> \
--cluster <cluster name>
```

## Adding PostgreSQL query analytics service

Use the `postgresql:queries` alias to enable PostgreSQL query analytics.

### USAGE

```
ssm-admin add postgresql:queries [NAME] [OPTIONS]
```

This creates the `ssm-postgresql-queries` service that is able to collect QAN data for multiple remote PostgreSQL server instances.

The **ssm-admin add** command is able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following options can be used with the `postgresql:queries` alias:

`--create-user`
: Create a dedicated PostgreSQL user for SSM Client (named `ssm`).

`--create-user-password`
: Specify password for the dedicated PostgreSQL user.

`--force`
:     Force to create or update the dedicated PostgreSQL user.

`--host`
:     Specify the PostgreSQL host name.

`--password`
:     Specify the password for PostgreSQL user with admin privileges.

`--port`
:     Specify the PostgreSQL instance port.

`--query-source`
:     Specify the source of data:

    * `auto`: Select automatically (default).
    * `logfile`: Use the log file.
    * `table`: Use table.

`--user`
: Specify the name of PostgreSQL user with admin privileges.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

### DETAILED DESCRIPTION

When adding the PostgreSQL query analytics service, the **ssm-admin** tool will attempt to automatically detect the local PostgreSQL instance and PostgreSQL superuser credentials.  You can use options to provide this information, if it cannot be detected automatically.

You can also specify the `--create-user` option to create a dedicated `ssm` user on the PostgreSQL instance that you want to monitor. This user will be given all the necessary privileges for monitoring, and is recommended over using the PostgreSQL superuser.

For example, to set up remote monitoring of QAN data on a PostgreSQL server located at 192.168.200.2, use a command similar to the following:

```
$ ssm-admin add postgresql:queries --user root --password root --host 192.168.200.2 --create-user
```

QAN can use either the *Log file* or *Table* as the source. By default, it chooses the *Log file* for a local PostgreSQL instance and *Table* otherwise. For more information about the differences, see [Configuring Query Analytics](conf-postgres.md#configuring-query-analytics).

You can explicitly set the query source when adding a QAN instance using the `--query-source` option.

For more information, run **ssm-admin add** `postgresql:queries --help`.

## Adding PostgreSQL metrics service

Use the `postgresql:metrics` alias to enable PostgreSQL metrics monitoring.

### USAGE

```
$ ssm-admin add postgresql:metrics [NAME] [OPTIONS]
```

This creates the `ssm-postgresql-metrics` service that collects PostgreSQL instance metrics.

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following options can be used with the `postgresql:metrics` alias:

`--create-user`
: Create a dedicated PostgreSQL user for SSM Client (named `ssm`).

`--create-user-password`
: Specify password for the dedicated PostgreSQL user.

`--force`
: Force to create or update the dedicated PostgreSQL user.

`--host`
: Specify the PostgreSQL host name.

`--password`
: Specify the password for PostgreSQL user with admin privileges.

`--port`
: Specify the PostgreSQL instance port.

`--socket`
: Specify the PostgreSQL instance socket file.

`--user`
: Specify the name of PostgreSQL user with admin privileges.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

### DETAILED DESCRIPTION

When adding the PostgreSQL metrics monitoring service, the **ssm-admin** tool attempts to automatically detect the local PostgreSQL instance and PostgreSQL superuser credentials.  You can use options to provide this information, if it cannot be detected automatically.

You can also specify the `--create-user` option to create a dedicated `ssm` user on the PostgreSQL host that you want to monitor.  This user will be given all the necessary privileges for monitoring, and is recommended over using the PostgreSQL superuser.

For example, to set up remote monitoring of PostgreSQL metrics on a server located at 192.168.200.3, use a command similar to the following:

```
$ ssm-admin add postgresql:metrics --user root --password root --host 192.168.200.3 --create-user
```

For more information, run **ssm-admin add** `postgresql:metrics` `--help`.


## Adding ProxySQL metrics service

Use the `proxysql:metrics` alias to enable ProxySQL performance metrics monitoring.

### USAGE

```
$ ssm-admin add proxysql:metrics [NAME] [OPTIONS]
```

This creates the `ssm-proxysql-metrics-42004` service that collects local ProxySQL performance metrics.

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following option can be used with the `proxysql:metrics` alias:

`--dsn`
: Specify the ProxySQL connection DSN. By default, it is `stats:stats@tcp(localhost:6032)/`.

You can also use global options that apply to any other command, as well as options that apply to adding services in general.

For more information, run **ssm-admin add** `proxysql:metrics` `--help`.

## Adding annotations

Use the **ssm-admin annotate** command to set notifications about important application events and display them on all dashboards. By using annotations, you can conveniently analyze the impact of application events on your database.

### USAGE

Run this command as root or by using the **sudo** command

```
$ ssm-admin annotate "Upgrade to v1.2" --tags "UX Imrovement,v1.2"
```

### OPTIONS

The **ssm-admin annotate** supports the following options:

`--tags`
: Specify one or more tags applicable to the annotation that you are creating. Enclose your tags in quotes and separate individual tags by a comma, such as “tag 1,tag 2”.

You can also use global options that apply to any other command.

## Checking network connectivity

Use the **ssm-admin check-network** command to run tests that verify connectivity between SSM Client and SSM Server.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin check-network [OPTIONS]
```

### OPTIONS

The **ssm-admin check-network** command does not have its own options, but you can use global options that apply to any other command

### DETAILED DESCRIPTION

Connection tests are performed both ways, with results separated accordingly:

* `Client --> Server`

    Pings Consul API, *SSM Query Analytics* API, and Prometheus API to make sure they are alive and reachable.

    Performs a connection performance test to see the latency from SSM Client to SSM Server.

* `Client <-- Server`

    Checks the status of Prometheus endpoints and makes sure it can scrape metrics from corresponding exporters.

    Successful pings of SSM Server from SSM Client do not mean that Prometheus is able to scrape from exporters. If the output shows some endpoints in problem state, make sure that the corresponding service is running (see **ssm-admin list**). If the services that correspond to problematic endpoints are running, make sure that firewall settings on the SSM Client host allow incoming connections for corresponding ports.

### OUTPUT EXAMPLE

```
$ ssm-admin check-network
SSM Network Status

Server Address | 192.168.100.1
Client Address | 192.168.200.1

* System Time
NTP Server (0.pool.ntp.org)         | 2017-05-03 12:05:38 -0400 EDT
SSM Server                          | 2017-05-03 16:05:38 +0000 GMT
SSM Client                          | 2017-05-03 12:05:38 -0400 EDT
SSM Server Time Drift               | OK
SSM Client Time Drift               | OK
SSM Client to SSM Server Time Drift | OK

* Connection: Client --> Server
-------------------- -------------
SERVER SERVICE       STATUS
-------------------- -------------
Consul API           OK
Prometheus API       OK
Query Analytics API  OK

Connection duration | 166.689µs
Request duration    | 364.527µs
Full round trip     | 531.216µs

* Connection: Client <-- Server
---------------- ----------- -------------------- -------- ---------- ---------
SERVICE TYPE     NAME        REMOTE ENDPOINT      STATUS   HTTPS/TLS  PASSWORD
---------------- ----------- -------------------- -------- ---------- ---------
linux:metrics    mongo-main  192.168.200.1:42000  OK       YES        -
mongodb:metrics  mongo-main  192.168.200.1:42003  PROBLEM  YES        -
```

For more information, run **ssm-admin check-network** `--help`.

## Obtaining Diagnostics Data for Support

SSM Client is able to generate a set of files for enhanced diagnostics, which is designed to be shared with Shattered Silicon Support to solve an issue faster. This feature fetches logs, network, and the Percona Toolkit output. To perform data collection by SSM Client, execute:

```
ssm-admin summary
```

The output will be a tarball you can examine. The single file will look like this:

```
summary__2018_10_10_16_20_00.tar.gz
```

## Configuring SSM Client

Use the **ssm-admin config** command to configure how SSM Client communicates with SSM Server.

### USAGE

Run this command as root or by using the **sudo** command.

```
ssm-admin config [OPTIONS]
```

### OPTIONS

The following options can be used with the **ssm-admin config** command:

`--bind-address`
: Specify the bind address, which is also the local (private) address mapped from client address via NAT or port forwarding By default, it is set to the client address.

`--client-address`
: Specify the client address, which is also the remote (public) address for this system. By default, it is automatically detected via request to server.

`--client-name`
: Specify the client name. By default, it is set to the host name.

`--force`
: Force to set the client name on initial setup after uninstall with unreachable server.

`--server`
: Specify the address of the SSM Server host. If necessary, you can also specify the port after colon, for example:

    ```
    ssm-admin config --server 192.168.100.6:8080
    ```

    By default, port 80 is used with SSL disabled, and port 443 when SSL is enabled.

`--server-insecure-ssl`
: Enable insecure SSL (self-signed certificate).

`--server-password`
: Specify the HTTP password configured on SSM Server.

`--server-ssl`
: Enable SSL encryption for connection to SSM Server.

`--server-user`
: Specify the HTTP user configured on SSM Server (default is `ssm`).

You can also use global options that apply to any other command.

For more information, run **ssm-admin config** –help.

## Getting help for any command

Use the **ssm-admin help** command to print help for any command.

### USAGE

Run this command as root or by using the **sudo** command

```
$ ssm-admin help [COMMAND]
```

This will print help information and exit.  The actual command is not run and options are ignored.

!!! alert alert-info "Note"
    You can also use the global `-h` or `--help` option after any command to get the same help information.

### COMMANDS

You can print help information for any command or service alias.

## Getting information about SSM Client

Use the **ssm-admin info** command to print basic information about SSM Client.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin info [OPTIONS]
```

### OPTIONS

The **ssm-admin info** command does not have its own options, but you can use global options that apply to any other command

### OUTPUT

The output provides the following information:

* Version of **ssm-admin**
* SSM Server host address, and local host name and address (this can be configured using **ssm-admin config**)
* System manager that **ssm-admin** uses to manage SSM services
* Go version and runtime information

For example:

```
$ ssm-admin info

SSM Server      | 192.168.100.1
Client Name     | ubuntu-amd64
Client Address  | 192.168.200.1
Service manager | linux-systemd

Go Version      | 1.8
Runtime Info    | linux/amd64
```

For more information, run **ssm-admin info** `--help`.

## Listing monitoring services

Use the **ssm-admin list** command to list all enabled services with details.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin list [OPTIONS]
```

### OPTIONS

The **ssm-admin list** command supports global options that apply to any other command and also provides a machine friendly JSON output.

`--json`
: list the enabled services as a JSON document. The information provided in the standard tabular form is captured as keys and values. The general information about the computer where SSM Client is installed is given as top level elements:

    * `Version`
    * `ServerAddress`
    * `ServerSecurity`
    * `ClientName`
    * `ClientAddress`
    * `ClientBindAddress`
    * `Platform`

    Note that you can quickly determine if there are any errors by inspecting the `Err` top level element in the JSON output. Similarly, the `ExternalErr` element reports errors in external services.

    The `Services` top level element contains a list of documents which represent enabled monitoring services. Each attribute in a document maps to the column in the tabular output.

    The `ExternalServices` element contains a list of documents which represent enabled external monitoring services. Each attribute in a document maps to the column in the tabular output.

### OUTPUT

The output provides the following information:

* Version of **ssm-admin**
* SSM Server host address, and local host name and address (this can be configured using **ssm-admin config**)
* System manager that **ssm-admin** uses to manage SSM services
* A table that lists all services currently managed by `ssm-admin`, with basic information about each service

For example, if you enable general OS and MongoDB metrics monitoring, output should be similar to the following:

Run this command as root or by using the **sudo** command

```
$ ssm-admin list

...

SSM Server      | 192.168.100.1
Client Name     | ubuntu-amd64
Client Address  | 192.168.200.1
Service manager | linux-systemd

---------------- ----------- ----------- -------- ---------------- --------
SERVICE TYPE     NAME        LOCAL PORT  RUNNING  DATA SOURCE      OPTIONS
---------------- ----------- ----------- -------- ---------------- --------
linux:metrics    mongo-main  42000       YES      -
mongodb:metrics  mongo-main  42003       YES      localhost:27017
```

## Pinging SSM Server

Use the **ssm-admin ping** command to verify connectivity with SSM Server.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin ping [OPTIONS]
```

If the ping is successful, it returns `OK`.

```
$ ssm-admin ping
OK, SSM server is alive.

SSM Server      | 192.168.100.1 (insecure SSL, password-protected)
Client Name     | centos7.vm
Client Address  | 192.168.200.1
```

### OPTIONS

The **ssm-admin ping** command does not have its own options, but you can use global options that apply to any other command.

For more information, run **ssm-admin ping** `--help`.

## Purging metrics data

Use the **ssm-admin purge** command to purge metrics data associated with a service on SSM Server. This is usually required after you remove a service and do not want its metrics data to show up on graphs.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin purge [SERVICE [NAME]] [OPTIONS]
```

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### SERVICES

Specify a monitoring service alias. To see which services are enabled, run **ssm-admin list**.

### OPTIONS

The **ssm-admin purge** command does not have its own options, but you can use global options that apply to any other command

For more information, run **ssm-admin purge** `--help`.

## Removing monitoring services

Use the **ssm-admin rm** command to remove monitoring services.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin rm [OPTIONS] [SERVICE]
```

When you remove a service, collected data remains in Metrics Monitor on SSM Server. To remove the collected data, use the **ssm-admin purge** command.

### OPTIONS

The following option can be used with the **ssm-admin rm** command:

`--all`
: Remove all monitoring services.

You can also use global options that apply to any other command.

### SERVICES

Specify a monitoring service alias. To see which services are enabled, run **ssm-admin list**.

### EXAMPLES

* To remove all services enabled for this SSM Client:

    ```
    $ ssm-admin rm --all
    ```

* To remove all services related to MySQL:

    ```
    $ ssm-admin rm mysql
    ```

* To remove only `mongodb:metrics` service:

    ```
    $ ssm-admin rm mongodb:metrics
    ```

For more information, run **ssm-admin rm** –help.

## Removing orphaned services

Use the **ssm-admin repair** command to remove information about orphaned services from SSM Server. This can happen if you removed services locally while SSM Server was not available (disconnected or shut down), for example, using the **ssm-admin uninstall** command.

### USAGE

Run this command as root or by using the **sudo** command

```
$ ssm-admin repair [OPTIONS]
```

### OPTIONS

The **ssm-admin repair** command does not have its own options, but you can use global options that apply to any other command.

For more information, run **ssm-admin repair** –help.

## Restarting monitoring services

Use the **ssm-admin restart** command to restart services managed by this SSM Client. This is the same as running **ssm-admin stop** and **ssm-admin start**.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin restart [SERVICE [NAME]] [OPTIONS]
```

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following option can be used with the **ssm-admin restart** command:

`--all`

    Restart all monitoring services.

You can also use global options that apply to any other command.

### SERVICES

Specify a monitoring service alias that you want to restart. To see which services are available, run **ssm-admin list**.

### EXAMPLES

* To restart all available services for this SSM Client:

    ```
    # ssm-admin restart --all
    ```

* To restart all services related to MySQL:

    ```
    $ ssm-admin restart mysql
    ```

* To restart only the `mongodb:metrics` service:

    ```
    $ ssm-admin restart mongodb:metrics
    ```

For more information, run **ssm-admin restart** `--help`.

## Getting passwords used by SSM Client

Use the **ssm-admin show-passwords** command to print credentials stored in the configuration file (by default: `/opt/ss/ssm-client/ssm.yml`).

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin show-passwords [OPTIONS]
```

### OPTIONS

The **ssm-admin show-passwords** command does not have its own options, but you can use global options that apply to any other command

### OUTPUT

This command prints HTTP authentication credentials and the password for the `ssm` user that is created on the MySQL instance if you specify the `--create-user` option when adding a service.

Run this command as root or by using the **sudo** command

```
$ ssm-admin show-passwords
HTTP basic authentication
User     | aname
Password | secr3tPASS

MySQL new user creation
Password | g,3i-QR50tQJi9M1yl9-
```

For more information, run **ssm-admin show-passwords**  `--help`.

## Starting monitoring services

Use the **ssm-admin start** command to start services managed by this SSM Client.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin start [SERVICE [NAME]] [OPTIONS]
```

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following option can be used with the **ssm-admin start** command:

`--all`
: Start all monitoring services.

You can also use global options that apply to any other command.

### SERVICES

Specify a monitoring service alias that you want to start. To see which services are available, run **ssm-admin list**.

### EXAMPLES

* To start all available services for this SSM Client:

    ```
    $ ssm-admin start --all
    ```

* To start all services related to MySQL:

    ```
    $ ssm-admin start mysql
    ```

* To start only the `mongodb:metrics` service:

    ```
    $ ssm-admin start mongodb:metrics
    ```

For more information, run **ssm-admin start** `--help`.

## Stopping monitoring services

Use the **ssm-admin stop** command to stop services managed by this SSM Client.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin stop [SERVICE [NAME]] [OPTIONS]
```

!!! alert alert-info "Note"
    It should be able to detect the local SSM Client name, but you can also specify it explicitly as an argument.

### OPTIONS

The following option can be used with the **ssm-admin stop** command:

`--all`
: Stop all monitoring services.

You can also use global options that apply to any other command.

### SERVICES

Specify a monitoring service alias that you want to stop. To see which services are available, run **ssm-admin list**.

### EXAMPLES

* To stop all available services for this SSM Client:

    ```
    $ ssm-admin stop --all
    ```

* To stop all services related to MySQL:

    ```
    $ ssm-admin stop mysql
    ```

* To stop only the `mongodb:metrics` service:

    ```
    $ ssm-admin stop mongodb:metrics
    ```

For more information, run **ssm-admin stop** `--help`.

## Cleaning Up Before Uninstall

Use the **ssm-admin uninstall** command to remove all services even if SSM Server is not available.  To uninstall SSM correctly, you first need to remove all services, then uninstall SSM Client, and then stop and remove SSM Server.  However, if SSM Server is not available (disconnected or shut down), **ssm-admin rm** will not work.  In this case, you can use **ssm-admin uninstall** to force the removal of monitoring services enabled for SSM Client.

!!! alert alert-info "Note"
    Information about services will remain in SSM Server, and it will not let you add those services again.  To remove information about orphaned services from SSM Server, once it is back up and available to SSM Client, use the **ssm-admin repair** command.

### USAGE

Run this command as root or by using the **sudo** command

```
ssm-admin uninstall [OPTIONS]
```

### OPTIONS

The **ssm-admin uninstall** command does not have its own options, but you can use global options that apply to any other command.

For more information, run **ssm-admin uninstall** `--help`.

## Monitoring Service Aliases

The following aliases are used to designate SSM services that you want to add, remove, restart, start, or stop:

| Alias              | Services                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------- |
| `linux:metrics`    | General system metrics monitoring service                                                   |
| `mysql:metrics`    | MySQL metrics monitoring service                                                            |
| `mysql:queries`    | MySQL query analytics service                                                               |
| `mongodb:metrics`  | MongoDB metrics monitoring service                                                          |
| `mongodb:queries`  | MongoDB query analytics service                                                             |
| `proxysql:metrics` | ProxySQL metrics monitoring service                                                         |
| `mysql`            | Complete MySQL instance monitoring: `linux:metrics`, `mysql:metrics`, `mysql:queries`       |
| `mongodb`          | Complete MongoDB instance monitoring: `linux:metrics`, `mongodb:metrics`, `mongodb:queries` |
