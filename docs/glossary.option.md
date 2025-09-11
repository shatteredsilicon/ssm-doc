# SSM Server Additional Options

This glossary contains the additional parameters that you may pass when starting SSM Server.

## Passing options to SSM Server docker at first run

Docker allows configuration options to be passed using the flag `-e` followed by the option you would like to set.

Here, we pass more than one option to SSM Server along with the **docker run** command. Run this command as root or by using the **sudo** command.

```
$ docker run -d -p 80:80 \
  --volumes-from ssm-data \
  --name ssm-server \
  -e SERVER_USER=jsmith \
  -e SERVER_PASSWORD=SomeR4ndom-Pa$$w0rd \
  --restart always \
  shatteredsilicon/ssm-server:latest
```

## Passing options to *SSM Server* for an already deployed docker instance

Docker doesn’t support changing environment variables on an already provisioned container, therefore you need to stop the current container and start a new container with the new options. variable with **docker start** if you want to change the setting for existing installation, because **docker start** cares to keep container immutable and doesn’t support changing environment variables. Therefore if you want container with different properties,  you should run a new container instead.

1. Stop and Rename the old container:

    ```
    docker stop ssm-server
    docker rename ssm-server ssm-server-old
    ```

2. Ensure you are running the latest version of SSM Server:

    ```
    docker pull shatteredsilicon/ssm-server:latest
    ```

    !!! alert alert-warning "Warning"
        When you destroy and recreate the container, all the updates you have done through SSM Web interface will be lost. What’s more, the software version will be reset to the one in the Docker image. Running an old SSM version with a data volume modified by a new SSM version may cause unpredictable results. This could include data loss.

3. Start the container with the new settings. For example, changing `METRICS_RESOLUTION` would look as follows:

    ```
    docker run -d \
      -p 80:80 \
      --volumes-from ssm-data \
      --name ssm-server \
      --restart always \
      -e METRICS_RESOLUTION=5s \
      shatteredsilicon/ssm-server:latest
    ```

4. Once you’re satisfied with the new container deployment options and you don’t plan to revert, you can remove the old container:

    ```
    docker rm ssm-server-old
    ```

## List of SSM Server Parameters

### METRICS_RETENTION

This option determines how long metrics are stored at SSM Server. The value is passed as a combination of hours, minutes, and seconds, such as **720h0m0s**. The minutes (a number followed by *m*) and seconds (a number followed by *s*) are optional.

To set the `METRICS_RETENTION` option to 8 days, set this option to *192h*.

Run this command as root or by using the **sudo** command

```
$ docker run ... -e METRICS_RETENTION=192h ... shatteredsilicon-ssm:latest
```

### QUERIES_RETENTION

This option determines how many days queries are stored at SSM Server.

```
$ docker run ... -e QUERIES_RETENTION=30 ... shatteredsilicon-ssm:latest
```

### SSM_SQL_CHECK_TIMEOUT

This option overrides the default monitoring query timeout, particularly for the case where adding remote hosts could time out (e.g. due to extremely large number of tables, in the hundreds of thousands range, where SELECT COUNT(*) FROM information_schema.TABLES takes a long time). Default is 5s, which should be sufficient for most reasonable deployments.

```
$ docker run ... -e SSM_SQL_CHECK_TIMEOUT=10s ... shatteredsilicon-ssm:latest
```

### SSM_DISABLE_TABLESTATS_LIMIT

This option specifies the maximum number of tables for which collection of table statistics is enabled for remote MySQL instances/RDS instances (by default, the limit is 1 000 tables). Table statistics includes `auto_increment.columns`, `info_schema.tables`, `info_schema.tablestats`, `perf_schema.indexiowaits`, `perf_schema.tableiowaits`, `perf_schema.tablelocks` and `perf_schema.eventswaits`.

```
$ docker run ... -e SSM_DISABLE_TABLESTATS_LIMIT=512 ... shatteredsilicon-ssm:latest
```

### ORCHESTRATOR_ENABLED

This option enables Orchestrator. By default it is disabled. It is also disabled if this option contains **false**.

```
$ docker run ... -e ORCHESTRATOR_ENABLED=true ... shatteredsilicon-ssm:latest
```

### ORCHESTRATOR_USER

Pass this option, when running your SSM Server via Docker to set the orchestrator user. You only need this parameter (along with `ORCHESTRATOR_PASSWORD` if you have set up a custom Orchestrator user.

This option has no effect if the `ORCHESTRATOR_ENABLED` option is set to **false**.

```
$ docker run ... -e ORCHESTRATOR_ENABLED=true ORCHESTRATOR_USER=name -e ORCHESTRATOR_PASSWORD=pass ... shatteredsilicon-ssm:latest
```

### ORCHESTRATOR_PASSWORD

Pass this option, when running your SSM Server via Docker to set the orchestrator password.

This option has no effect if the `ORCHESTRATOR_ENABLED` option is set to **false**.

```
$ docker run ... -e ORCHESTRATOR_ENABLED=true ORCHESTRATOR_USER=name -e ORCHESTRATOR_PASSWORD=pass ... shatteredsilicon-ssm:latest
```

### SERVER_USER

By default, the user name is `ssm`. Use this option to use another user name.

Run this command as root or by using the **sudo** command.

```
$ docker run ... -e SERVER_USER=USER_NAME ... shatteredsilicon-ssm:latest
```

### SERVER_PASSWORD

Set the password to access the SSM Server web interface.

Run this command as root or by using the **sudo** command.

```
$ docker run ... -e SERVER_PASSWORD=YOUR_PASSWORD ... shatteredsilicon-ssm:latest
```

By default, the user name is `ssm`. You can change it by passing the `SERVER_USER` variable.

### METRICS_RESOLUTION

This environment variable sets the minimum resolution for checking metrics. You should set it if the latency is higher than 1 second.

Run this command as root or by using the **sudo** command.

```
$ docker run ... -e METRICS_RESOLUTION=VALUE ... shatteredsilicon-ssm:latest
```

### METRICS_MEMORY

By default, Prometheus in SSM Server uses all available memory for storing the most recently used data chunks.  Depending on the amount of data coming into Prometheus, you may require to allow less memory consumption if it is needed for other processes.

!!! alert alert-warning "Important"

    Starting with version 1.13.0, SSM uses Prometheus 2. Due to optimizations in Prometheus, setting the metrics memory by passing the `METRICS_MEMORY` option is a deprecated practice.

If you are still using a version of SSM prior to 1.13 you might need to set the metrics memory by passing the `METRICS_MEMORY` environment variable along with the **docker run** command.

Run this command as root or by using the **sudo** command. The value must be passed in kilobytes. For example, to set the limit to 4 GB of memory run the following command:

```
$ docker run ... -e METRICS_MEMORY=4194304 ... shatteredsilicon-ssm:latest
```

### DISABLE_UPDATES

To update your SSM from web interface you only need to click the Update on the home page. The `DISABLE_UPDATES` option is useful if updating is not desirable. Set it to **true** when running SSM in the Docker container.

Run this command as root or by using the **sudo** command.

```
$ docker run ... -e DISABLE_UPDATES=true ... shatteredsilicon-ssm:latest
```

The `DISABLE_UPDATES` option removes the Update button from the interface and prevents the system from being updated manually.

### QUERY FILTERING - QAN_FILTER_OMIT

To set the filter for remote mysql instances/RDS instances that are added on the server side, you can set up the QAN_FILTER_OMIT environment variable when you create the runtime container like this:

```
$ docker run ... -e QAN_FILTER_OMIT=COMMIT,RESET,PING,PREPARE,ROLLBACK,SET ... shatteredsilicon/ssm-server:latest
```

This will set up the default filter for all the mysql instances/RDS instances added on the server side. After you have added the instance, you can go to the SSM Query Analytics Setting.

![Screenshot of SSM query analytics settings](_images/ssm_query_analytics_settings.png)
