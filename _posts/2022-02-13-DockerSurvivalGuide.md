---
keywords: docker
description: "You probably know what Docker is and how they are commonly used in practice, but are looking for a quick guide to navigate through the jungle. This article summarises the basics in Docker and Docker Compose to help you get started."
title: "Survival Guide for Docker & Docker Compose"
toc: false 
badges: true
comments: true
categories: [de-zoomcamp, software engineering]
image: "images/2022-02-13-DockerSurvivalGuide/cover.png"
layout: post
---


![cover_from_webbylab.png]({{ site.baseurl }}/images/2022-02-13-DockerSurvivalGuide/cover.png "extracted from WebbyLab")


## **1. Who Should Read This?**
---
You probably know what Docker is and how they are commonly used in practice, but are looking for a quick guide to navigate through the jungle. This article summarises the basics in Docker and Docker Compose to help you get started. 

This is originally a note for [Data Engineering Zoomcamp](https://github.com/DataTalksClub/data-engineering-zoomcamp) lesson 1, and over time I expanded it with additional references. Now I decided to package it as a stand-alone survival guide with a hope to help those in my situation, who intend to quickly get themselves up for this amazing tools.

## **2. Essential Docker Commands**
---
#### **`docker run <image-name>:<tag> <command>`**

The command does the following in order:
1. look up target image `<image-name>` of specified tag `<tag>` locally, otherwise download it from Docker Hub  
(if `<tag>` is not specified, it assumes the tag to be `latest`)
2. spin off a container from the downloaded image
3. execute `<command>` in the running container, if provided
4. gracefully stop the container after the `<command>` finishes

Remarks:
- each execution of the command spins off independent container
- absolute path should be used if a path is needed in `docker run` 

![docker_run.png]({{ site.baseurl }}/images/2022-02-13-DockerSurvivalGuide/docker_run.png)

**Highlighted Flags**
<style>
table th:first-of-type {
    width: 20%;
}
table th:nth-of-type(2) {
    width: 80%;
}
</style>

| Flag | Description |
| --- | --- |
| `--detach` (or `-d`) | run the container in background |
| `--rm` | delete the container at the end of the command. useful if you intend to run the container once only |
| `--name` | specify a name to your container. useful for inter-container communication because other containers could address your container by its name instead of its IP address. |
| `--env` (or `-e`) | set an environment variable in the container. useful when the service to be containerized requires environment variables, repeatedly add `--env` for multiple environment variables. |
| `-it` | attach to the session in the container at the end and enable you to interact with the session, useful when you have appended command and it |
| `--label` (or `-l`) | add a meta label to the container. it’s useful if you want to group containers by labels, same as `--env` you can repeatedly add `--label` for multiple meta labels |
| `--volume` (or `-v`) | mount a directory in localhost to a directory in the container <br/> e.g. `--volume <local-directory>:<container-directory>` |
| `--expose` (or `-e`) | binds a port in local host to a port in container. useful for localhost to communicate with the containerized service <br/> e.g. `--expose <local-port>:<container-port>` |
| `--entrypoint` | run specified commands in the container, functionally similar to `<command>` in this case. |


**Examples**  
- `docker run python:3.8 ls`: spin off a container from image `python:3.8` and then list the folders & files at default directory of the container
- `docker run --rm -d ython:3.8 python -m http.server`: start a http server at default directory of a container and run the container in background (running in background is useful here, otherwise your terminal session can't resume unless you forcefully shut down the server). upon shutting down the container, it is automatically deleted (thanks to `--rm`).
- `docker run -it ubuntu bash`: spin off a container from `ubuntu:latest` image. execute `bash` in the container and then attach to the associated bash session (thanks to `-it`).
- `docker run -it -e POSTGRES_USER="root" -e POSTGRES_PASSWORD="root" -e POSTGRES_DB="ny_taxi" -p 5432:5432 postgres:13`: spin off a container from `postgres:13` image and set 3 environment variables for the postgres database in the container. additionally the container exposes its port `5432` to local host's port `5432`. then the container starts postgres server and we attach to the associated session.


#### **`docker container ls`**

This command lists out attributes for each container, by default it lists out running containers only. It is equivalent to `docker ps`.

![docker_ls.png]({{ site.baseurl }}/images/2022-02-13-DockerSurvivalGuide/docker_ls.png)

**Highlighted Flags**

| Flag | Description |
| --- | --- |
| `--filter` (or `-f`) | lists out containers that satisfy the specified filter. you need to specify key and (optionally) value for filtering. add multiple `--filter` flags for multiple conditions |
| `--all` (or `-a`) | lists out all containers (both stopped and running containers) |
| `--quiet` (or `-q`) | returns container IDs only. particularly useful when you want to propagate a list of container ID to another command |
| `--format` | customizes the attributes you want to print out for the containers. |
| `--size` | additionally show the file size for each containers. attribute `Size` is exposed for you to specify in `--format`. it is set as an additional flag because it is costly to query file size for each container. |

**Examples**
- {% raw %} `docker container ls --all ---size --format "ID:{{.ID}} Image:{{.Image}} Size:{{.Size}}"` {% endraw %}: print out all containers' IDs, image names and file sizes. note that {% raw %} `{{.Size}}` {% endraw %} is only accessible with `--size` flag
- `docker container ls --all --filter "ancestor=ubuntu" --filter "ancestor=python:3.8 --filter "label=version=1.3" -q`: print out ID of all containers whose image is `ubuntu:latest` OR whose image is `python:3.8` OR whose meta labels contain key `version` of value `1.3`.


#### **`docker container inspect <container-id>`**

You can pass one or more container names/ IDs to the command and it returns all metadata associated to those containers

![docker_inspect.png]({{ site.baseurl }}/images/2022-02-13-DockerSurvivalGuide/docker_inspect.png)

**Highlighted Flags**

| Flag | Description |
| --- | --- |
| `--format` (or `-f`) | functionally similar to `--format` from `docker container ls`, except you can access more attributes here. <br/> (e.g. `NetworkSettings.Networks.bridge.IPAddress` for container’s IP address) |
| `--size` | additionally expose attribute `SizeRootFs` in the metadata. it expresses container’s file size in bytes. |

**Examples**
- {% raw %} `docker inspect --size --format "{{.ID}} {{.Config.Image}} {{.SizeRootFs}} {{.NetworkSettings.Networks.bridge.IPAddress}}" $(docker ps -a -q)` {% endraw %}: print out all containers' IDs, associated image names, file sizes (in bytes) and IP addresses. you can't access `SizeRootFs` attribute without `--size` flag

#### **`docker exec <container-id> <command>`**
Execute a command `<command>` in a running container `<container-id>`

![docker_exec.png]({{ site.baseurl }}/images/2022-02-13-DockerSurvivalGuide/docker_exec.png)

**Highlighted Flags**

| Flag | Description |
| --- | --- |
| `--it` | attach to and interact with the session in the container after the command execution <br/> adasdasds |
| `--user` (or `-u`) | execute the command as a specified user (username or UID) |

**Examples**
- `docker exec -it -u 0 <container-id> bash`: initiate a bash session as root user (UID 0) in a running container as root user
- `docker exec <container-id> pwd`: print out the default directory of a running container

### **Some Other Useful Docker Commands**
#### `docker start <container-id>`
>activate a stopped container `<container-id>` to run again  

#### `docker stop <container-id>`
> gracefully shut down a running container `<container-id>`  

#### `docker kill <container-id>`
> forcefully shut down a running container `<container-id>`  

#### `docker logs <container-id>`
> fetch the latest logs from `<container-id>`'s terminal for the command that it is running  

#### `docker cp <src-path> <container-id>:<dest-path>`
> transfer file from local host `<src-path>` to a directory in container `<dest-path>`. to transfer a file from container to local host, use `docker cp <container-id>:<src-path> <dest-path>` instead


#### `docker container rm <container-id>`
> Delete the stopped container `<container-id>`. add `--force` (or `-f`) to forcefully stop a running container and then delete it.  
> e.g. `docker rm $(docker container ls --all --filter "ancestor=ubuntu" --quiet)` deletes all containers from the image `ubuntu:latest`


## **3. Customise Your Image with `Dockerfile`**
---
Sometimes you may want to containerisze your own service. If you don't find any base images from Docker Hub with all required dependencies pre-intsalled, you can create your custom image.

The workflow for creating and running a custom image is straight forward. You start by writing your own `Dockerfile`. The file contains all configurations required to set up your container (e.g. commands to install required depencies, mounting, commands to start up the service ... etc.). Once the `Dockerfile` is ready, you can build a custom image from your `Dockerfile`, and then spin off containers from it.


![docker pipeline]({{ site.baseurl }}/images/2022-02-13-DockerSurvivalGuide/dockerfile.jpeg "extracted from Pinterest")

**Step 1: Configure Your `Dockerfile`**

Here are the instructions you can specified in `Dockerfile` (some have their equivalent flags from `docker run`). Note that you can access environment variables by `$<env_var>` (or `$(<env_var>)`) in `Dockerfile`:

| Instruction | Description |
| --- | --- |
| `FROM` | base image that you want to build upon, you can specify a specific tag for the image <br/> (e.g. `FROM python:3.8`) |
| `RUN` | command you want to run in the container, usually for commands that install dependencies <br/> (e.g. `RUN pip install -r requirements.txt`) |
| `WORKDIR` | set a working directory for any instructions that follow. if relative path is used, it will be relative to the `WORKDIR` from latest instruction |
| `COPY` | copy files or directory from local host to the container |
| `LABEL` | attach meta label to the contaienr, equivalent to `--label` (or `-l`) in `docker run` <br/> (e.g. `LABEL version=4.4` equivalent to `docker ps --filter label=version=4.4`) |
| `ENTRYPOINT` | default command you call in the container. it can’t be override by `docker run` |
| `CMD` | default command you call in the container, it can be overriden by the command appended to `docker run`. <br/> additionally when used with `ENTRYPOINT`, it serves as parameters to the command from `ENTRYPOINT` |

Here is an example for containerizing a data pipeline written in a Python script.
```docker
FROM sample-image:sample-tag

RUN pip install pandas
WORKDIR /working_dir
WORKDIR working_subdir
COPY pipeline.py pipeline.py
LABEL version="1.3"
LABEL create_date="2022-02-01"

ENTRYPOINT ["python", "pipeline.py"]
CMD ["2022-02-01"]
```

**Step 2: Build an image from your `Dockerfile`**

Build a custom image of name `<image-name>` with tag `<tag>` from your `Dockerfile`:
```shell
docker build -t <image-name>:<tag> .
```
- note that rerunning the command overwrites the previous image
- `.` means reading `Dockerfile` in current directory
- to check the image has been successfully built, do `docker images` to list out available images  

**Step 3: Fire off a container from your built image**

Once the image is built, you can spin off a container from it:
```shell
docker run <image-name>:<tag>
```

## **4. Manage Your Containers Easily with Docker Compose**
---

Sometimes it could be a pain to manage a group of containers with manual commands. Docker Compose is a tool to simplify the process. You can configure the services in `docker-compose.yml` file, and then you can easily manage them (e.g. build images, spin off/ stop containers) with `docker-compose` commands.

Let's say you want to separately containerize a Postgres server and a pgAdmin server (GUI tool for interacting with Postgres database). On top of that, the containerized pgAdmin should be able to connect with the containerized Postgres server and our local host should be able to access the containerized pgAdmin GUI. You can do the steps below to manage the containers.


**Step 1: Configure Your `docker-compose.yml`**

`docker-compose.yml` defines all configurations you want to set for each service. Note that unlike `docker run`, we can use relative path in `docker-compose.yml`.

```docker
services:
  pgdatabase:
    image: postgres:13
	environment:
	  - POSTGRES_USER=root
	  - POSTGRES_PASSWORD=root
	  - POSTGRES_DB=ny_taxi
	volumes:
	  - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
	ports:
	  - "5432:5432"
  pgadmin:
	image: dpage/pgadmin4
	environment:
	  - PGADMIN_DEFAULT_EMAIL=admin@gmail.com
	  - PGADMIN_DEFAULT_PASSWORD=root
	ports:
	  - "8080:80"
```


**Step 2: Bring Up Containers**  

Once `docker-compose.yml` is ready, you can run the following command in the same directory to bring up the containers. The command builds custom image and spin off container for each service:
```shell
docker-compose up
```
- similar to `docker run`, you can attach `--detach` (or `-d`) for running the group of containers in detached mode
- by default a custom bridge network will be created for the group of containers to enable inter-container communication by IP address or name resolution
- unlike `docker build`, rerunning `docker-compose up` doesn't rebuild an image even with updated `docker-compose.yaml`
- to reflect your change in `docker-compose.yaml`, you can either:
    - apply `docker-compose up --build` to enforce re-building, or equivalently
    - apply `docker-compose build` before `docker-compose up`
- note that the container's name will have additional prefix and suffix on top of what you specify in `docker-compose.yml`. the prefix is based on the name of the folder that you run `docker-compose up`. you can override the prefix by the flag `--project-name` or `-p` (i.e. `docker-compose --project-name <some-prefix> up`)


**Step 3: Bring Down Containers**  

Finally you can bring down what you have brought up from `docker-compose up` by:
```shell
docker-compose down
```
- not only it shuts down the containers, it will also remove the associated custom bridge network and the containers
- in case there are residual containers remained, you can apply `docker-compose rm` in the same directory to erase them

## **5. References**
---
1. [Youtube: DE Zoomcamp 1.2.1 - Introduction to Docker](https://www.youtube.com/watch?v=EYNwNlOrpr0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb)
2. [Docker Documentation: docker run](https://docs.docker.com/engine/reference/commandline/run/)
3. [Docker Documentation: docker container ls](https://docs.docker.com/engine/reference/commandline/container_ls/)
4. [Github Issue: Docker PS filter by ancestor image does not match tagged images if image tag is omitted from filter · Issue #24295 · moby/moby](https://github.com/moby/moby/issues/24295)
5. [Youtube: Manage Docker Easily With VS Code](https://www.youtube.com/watch?v=4I8CRAzPLD4)