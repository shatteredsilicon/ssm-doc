# Setting Up a Docker Container for SSM Server

[TOC]

A Docker image is a collection of preinstalled software which enables running a selected version of SSM Server on your computer. A Docker image is not run directly. You use it to create a Docker container for your SSM Server. When launched, the Docker container gives access to the whole functionality of SSM.

The setup begins with pulling the required Docker image. Then, you proceed by creating a special container for persistent SSM data. The last step is creating and launching the SSM Server container.

## Pulling the SSM Server Docker Image

To pull the latest version from Docker Hub:

```
$ docker pull shatteredsilicon/ssm-server:latest
```

This step is not required if you are running SSM Server for the first time. However, it ensures that if there is an older version of the image tagged with `latest` available locally, it will be replaced by the actual latest version.

## Creating the ssm-data Container

To create a container for persistent SSM data, run the following command:

```
$ docker create \
   -v /opt/prometheus/data \
   -v /opt/consul-data \
   -v /var/lib/mysql \
   -v /var/lib/grafana \
   --name ssm-data \
   shatteredsilicon/ssm-server:latest /bin/true
```

!!! alert alert-info "Note"
    This container does not run, it simply exists to make sure you retain all SSM data when you upgrade to a newer SSM Server image.  Do not remove or re-create this container, unless you intend to wipe out all SSM data and start over.

The previous command does the following:

* The **docker create** command instructs the Docker daemon to create a container from an image.
* The `-v` options initialize data volumes for the container.
* The `--name` option assigns a custom name for the container that you can use to reference the container within a Docker network. In this case: `ssm-data`.
* `shatteredsilicon/ssm-server:latest` is the name and version tag of the image to derive the container from.
* `/bin/true` is the command that the container runs.

!!! alert alert-warning "Important"
    Make sure that the data volumes that you initialize with the `-v` option match those given in the example. SSM Server expects that those directories are bind mounted exactly as demonstrated.

## Creating and Launching the SSM Server Container

To create and launch SSM Server in one command, use **docker run**:

```
$ docker run -d \
   -p 80:80 -p 443:443 \
   --volumes-from ssm-data \
   --name ssm-server \
   --restart always \
   shatteredsilicon/ssm-server:latest
```

This command does the following:

* The **docker run** command runs a new container based on the `shatteredsilicon/ssm-server:latest` image.
* The `-d` option starts the container in the background (detached mode).
* The `-p` option maps the port for accessing the SSM Server web UI. For example, if port **80** is not available, you can map the landing page to port 8080 using `-p 8080:80`.
* The `-v` option mounts volumes from the `ssm-data` container (see Creating the ssm-data Container).
* The `--name` option assigns a custom name to the container that you can use to reference the container within the Docker network. In this case: `ssm-server`.
* The `--restart` option defines the container’s restart policy. Setting it to `always` ensures that the Docker daemon will start the container on startup and restart it if the container exits.
* `shatteredsilicon/ssm-server:latest` is the name and version tag of the image to derive the container from.

## Installing and using specific docker version

To install specific SSM Server version instead of the latest one, just put desired version number after the colon. Also in this scenario it may be useful to [prevent updating SSM Server via the web interface](../../glossary.option.md) with the `DISABLE_UPDATES` docker option.

For example, installing version 9.4 with http auth username ssm and password "test" would look as follows:

```
$ docker create \
   -v /opt/prometheus/data \
   -v /opt/consul-data \
   -v /var/lib/mysql \
   -v /var/lib/grafana \
   --name ssm-data \
   shatteredsilicon/ssm-server:9.4 /bin/true

$ docker run -d \
   -p 80:80 -p 443:443 \
   --volumes-from ssm-data \
   --name ssm-server \
   -e SERVER_USER=ssm \
   -e SERVER_PASSWORD=test \
   --restart always \
   shatteredsilicon/ssm-server:9.4
```

## Additional options

When running the SSM Server, you may pass additional parameters to the **docker run** subcommand. All options that appear after the `-e` option are the additional parameters that modify the way how SSM Server operates.

The section [SSM Server Additional Options](../../glossary.option.md) lists all supported additional options.
