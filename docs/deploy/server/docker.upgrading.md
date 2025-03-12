# Updating SSM Server Using Docker

To check the version of SSM Server, run **docker ps** on the host.

Run the following commands as root or by using the **sudo** command

```
$ docker ps
CONTAINER ID   IMAGE                      COMMAND                CREATED       STATUS             PORTS                               NAMES
480696cd4187   shatteredsilicon/ssm-server:8.6.1.17.5.2-1   "/opt/entrypoint.sh"   4 weeks ago   Up About an hour   192.168.100.1:80->80/tcp, 443/tcp   ssm-server
```

The version number is visible in the Image column. For a Docker container created from the image tagged `latest`, the Image column contains `latest` and not the specific version number of SSM Server.

The information about the currently installed version of SSM Server is available from the `SSM_VERSION` environment variable. You may extract the version number by using the **docker exec** command:

```
$ docker exec -it ssm-server bash -c 'echo "$SSM_VERSION"'
# 8.7.1.17.5.2-1
```

To check if there exists a newer version of SSM Server, visit [shatteredsilicon/ssm-server](https://hub.docker.com/r/shatteredsilicon/ssm-server/tags/).

## Creating a backup version of the current ssm-server Docker container

You need to create a backup version of the current `ssm-server` container if the update procedure does not complete successfully or if you decide not to upgrade your SSM Server after trying the new version.

The **docker stop** command stops the currently running `ssm-server` container:

```
$ docker stop ssm-server
```

The following command simply renames the current `ssm-server` container to avoid name conflicts during the update procedure:

```
$ docker rename ssm-server ssm-server-backup
```

## Pulling a new Docker Image

Docker images for all versions of SSM are available from [shatteredsilicon/ssm-server](https://hub.docker.com/r/shatteredsilicon/ssm-server/tags/) Docker repository.

When pulling a newer Docker image, you may either use a specific version number or the `latest` image which always matches the highest version number.

This example shows how to pull a specific version:

```
$ docker pull shatteredsilicon/ssm-server:8.7.1.17.5.2-1
```

This example shows how to pull the `latest` version:

```
$ docker pull shatteredsilicon/ssm-server:latest
```

## Creating a new Docker container based on the new image

After you have pulled a new version of SSM from the Docker repository, you can use **docker run** to create a `ssm-server` container using the new image.

```
$ docker run -d \
   -p 80:80 \
   --volumes-from ssm-data \
   --name ssm-server \
   --restart always \
   shatteredsilicon/ssm-server:latest
```

!!! alert alert-warning "Important""
    The `ssm-server` container must be stopped before attempting **docker.run**.

The **docker run** command refers to the pulled image as the last parameter. If you used a specific version number when running **docker pull** (see [Pulling the SSM Server Docker Image](docker.setting-up.md)) replace `latest` accordingly.

Note that this command also refers to `ssm-data` as the value of `--volumes-from` option. This way, your new version will continue to use the existing data.

!!! alert alert-warning "Warning"
    Do not remove the `ssm-data` container when updating, if you want to keep all collected data.

Check if the new container is running using **docker ps**.

```
$ docker ps
CONTAINER ID   IMAGE                      COMMAND                CREATED         STATUS         PORTS                               NAMES
480696cd4187   shatteredsilicon/ssm-server:8.7.1.17.5.2-1   "/opt/entrypoint.sh"   4 minutes ago   Up 4 minutes   192.168.100.1:80->80/tcp, 443/tcp   ssm-server
```

Then, make sure that the SSM version has been updated (see [SSM Version](../../glossary.terminology.md#ssm-version)) by checking the SSM Server web interface.

!!! alert alert-warning "Warning"
    After an upgrade, in some cases it can take 5-10 minutes for the container to start due to the need to update on-disk data formatting of some components. This will usually prevent subsequent downgrades.

## Removing the backup container

After you have tried the features of the new version, you may decide to continue using it. The backup container that you have stored is no longer needed in this case.

To remove this backup container, you need the **docker rm** command:

```
$ docker rm ssm-server-backup
```

As the parameter to **docker rm**, supply the tag name of your backup container.

### Restoring the previous version

If, for whatever reason, you decide to keep using the old version, you just need to stop and remove the new `ssm-server` container.

```
$ docker stop ssm-server && docker rm ssm-server
```

Now, rename the `ssm-server-backup` to `ssm-server` and start it.

```
$ docker start ssm-server
```

!!! alert alert-warning "Warning"
    Do not use the **docker run** command to start the container. The **docker run** command creates and then runs a new container.

    To start a new container use the **docker start** command.
