# Docker Notes

Notes taken from reading how to use Docker online.

## Using an external file to set environment variables to a container

Using the `env_file` configuration option, you can pass multiple environment variables from an external file through to a service's container. It works like `docker run --env-file=FILE ...`.

In the `docker-compose.yml`, do the following setup

```docker
services:
    jupyter:
        ...
        env_file:
            - my_variable.env
```

The `my_variable.env` looks like this:

```bash
DEBUG=1
TAG=v1.4
```

## Map a local folder/directory to the container

The `%cd%` refers to the current directory/folder that you are in. And, map that to a directory inside the container. Recommendation is to navigate to the directory, then run the following command on Windows command prompt.

```command prompt
docker run --rm -it -v %cd%:/data <container-name>
```

If you are using Windows Powershell,

```powershell
docker run --rm -it -v ${PWD}:/data <container-name>
```

And, if you are using Linux or Mac OS,

```bash
docker run --rm -it -v `pwd`:/data <container-name>
```

## Setting up PostgreSQL on Windows 10 with Docker

Setting up a container running Postgres and expose the port used by Postgres to allow access from the host.

`docker run -p 5432:5432 --name yourContainerName -e POSTGRES_PASSWORD=yourPassword -d postgres`

The problem using above approach is the risk of losing data, if rebuilding the container is necessary. Therefore, it is advisable to use a secondary container to hold the data, separating it from the Postgres container.

The following command will create the data container using the Alpine image. You can read about this in the links under the *Reference Material*.

`docker create -v /var/lib/postgresql/data --name PostgresData alpine`

**Important note:** The -v parameter must match the path that Postgres expects.

Then, you can set up a container that uses the data container as storage:

`docker run -p 5432:5432 --name yourContainerName -e POSTGRES_PASSWORD=yourPassword -d --volumes-from PostgresData postgres`

### Removing the PostgreSQL container, including the associated volume

Technically, your data will not be lost unless you use the -v flag when you remove the container.

`docker rm -v yourContainerName`

If you neglect to include the -v flag when removing the container, the data will remain in a dangling volume and, while still present on your host’s file system, not very easy to work with and not automatically mounted when you start a new postgresql container.

### Potential Issues

1. This is an error I received on Apr 13, 2019. You might receive error that says

`docker: Error response from daemon: driver failed programming external connectivity on endpoint ...: Error starting userland proxy: mkdir /port/tcp:0.0.0.0:5432:tcp:172.17.0.2:5432: input/output error.`

You need to quite/close the Docker Desktop, and restart the Docker Desktop. Then, it will work.

### Reference Material

* [Setup PostgreSQL on Windows with Docker](http://elanderson.net/2018/02/setup-postgresql-on-windows-with-docker/)
* [Dockerized Postgresql Development Environment](http://ryaneschinger.com/blog/dockerized-postgresql-development-environment/)
* [Deploying PostgreSQL on a Docker Container](https://severalnines.com/database-blog/deploying-postgresql-docker-container)
* [Data Science at the command line](https://www.datascienceatthecommandline.com/)
