# Docker

> an open source project that automates the deployment of software applications
> inside *containers* by providing additional layers of abstraction and automation of *OS-level virtualization* on linux.

#### Base images vs Child images

- *Base images* are images that have no parent images, usually images with an OS like ubuntu, alpine or debian.
- *Child images* are images that build on base images and add additional functionality.

#### Docker vs Vagrant

[source](https://www.quora.com/What-is-the-difference-between-Docker-and-Vagrant-When-should-you-use-each-one)

Docker relies on `containerization` while Vagrant utilizes `virtualization`.

With `virtualization`, each virtual machine runs its own entire operating system
inside a simulated hardware environment provided by program called __hypervisor__ running on
the physical hardware.

Pros:
- near-complete separation between virtual machine and the host enables you to have linux virtual machine on
a windows host or vice versa.

Cons:
- you have dedicated a static amount of resources (CPU, RAM, storage) to the virtual machines, and
the hypervisor will eat up a lot of resources (this is usually referred to as overhead)

With `containerization` allows multiple applications to run in isolated partitions of a single linux kernel
running directly on physical hardware.

Pros:
- performance is higher than virtualization since there is no hypervisor overhead, and you are closer to the bare metal.
Also the container simply uses whatever resoucers its needed, period.

Cons:
- Containers use the host machine's kernel.

#### Docker terminology

- __Images__: The blueprints of our application which form the basis of containers. We used `docker pull` command to download the image.
- __Containers__: Created from Docker image and run the actual application. We create container using `docker run` with image we download with `docker pull` command. A list of running containers can be seen using the `docker ps` command.
- __Docker Daemon__: The background service running on the host that manages building, running and distributing Docker containers.
  The daemon is the process that runs in the operating system to which clients talk to.
- __Docker Client__: The command line tool that allows the user to interact with the daemon.
- __Docker Hub__: A registry of docker images. You can think of the registry as a directory of all available docker images.

#### Writing your Dockerfile

- `FROM`
  Define your base image, e.g `alpine`, `ubuntu`, etc.

- `LABEL`
  You can add label to your Dockerfile. e.g:
    ```
    LABEL com.example.version="1.5.0"
    LABEL your_label="LABEL"
    ```
- `RUN`
  Run or execute command inside docker. e.g `RUN apt-get install package-bar`.
  `RUN` keyword has 2 forms:

  - `RUN <command>`
    This is called __shell__, which by default is `/bin/sh -c` on linux and `cmd /S /C` on windows.

  - `RUN ["executable", "param1", "param2"]`
    This is __exec__ form

  Always combine your `RUN` command to minimize layers and split into multiple line if its long enough.

- `CMD`
  This command is similar with `RUN` but it should be to execute only the software contained by the docker image.
  `CMD` command has 3 forms:

  - `CMD ["executable", "param1", "param2"]`: __exec__ form
  - `CMD ["param1", "param2"]`: entry point
  - `CMD command param1 param2`: __shell__ form

  `CMD`
  Should always be used in exec form and there is must be only one `CMD` command in dockerfile.

- `EXPOSE`
  Expose specific ports on container.

- `ENV`
  Set environment variables. But it would be better to set env variables with `RUN` command.
  Instead:
    ```
    FROM alpine
    ENV ADMIN_USER="mark"
    RUN echo $ADMIN_USER > ./mark
    RUN unset ADMIN_USER
    CMD sh
    ```

  You can:
    ```
    FROM alpine
    RUN export ADMIN_USER="mark" \
        && echo $ADMIN_USER > ./mark \
        && unset ADMIN_USER
    CMD sh
    ```

- `ADD` & `COPY`
  These command has similar function. These command will copy new file into specific directory.
  Despite these command is similar, but `COPY` is more preferred.
  e.g:
    ```
    COPY requirements.txt /tmp/
    RUN pip install --requirement /tmp/requirements.txt
    COPY . /tmp/

    ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
    ADD test /absoluteDir/         # adds "test" to /absoluteDir/
    ```

- `WORKDIR`
  This command will sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` command
  that follow it in the dockerfile. At its best, `WORKDIR` should always use absolute paths.
  e.g:
    ```
    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd
    ```



#### Most used commands in Docker

- `docker --version`: check docker version.
- `docker pull`: download docker image.
- `docker inspect [image]`: get image details.
- `docker run`: create new container. To test is docker installation is complete you can run `docker run hello-world`.
  You may often use this command and combine it with flags:
  - `docker run -d -p 80:80 --name webserver package-name`:
    - `-d`/`--detach` flag make you run container in background and print container ID.
    - `-p`/`--publish` flag to publish your container with specific port(s).
    - `--name` flag to give your container a name.

  - `docker run -it [image] [command]`: will make you able to run multiple command on single container

- `docker info`: get details about your docker installation.
- `docker container ls --all` or `docker ps -a`: show all containers that we run. Remove `--all`/`-a` flag to see only running container.
- `docker container stop [name|ID]`: stop one or more running containers. You can use container `name` or `ID`.
- `docker container rm [name|ID]`: remove one or more containers.

- `docker service ls`: show all service.
- `docker service ps your_service`: show all tasks of your services.
- `docker image ls`: list out the image you have downloaded (in this example `hello-world` image).
- `docker container ls --all`: list out `hello-world` container.
- `docker ps`: get list containers that are running. Add `-a` option to list all containers that we run
- `docker rm $(docker ps -a -q -f status=exited)`: you can use this to remove all containers that has been running.
  This command delete all containers that have status `status=exited`. `-q` flag to return only the numeric IDs
  and `-f` filters output based on conditions provided.

- `docker stack rm your_stack_name`: shutdown your app.
- `docker swarm leave --force`: shutdown
- `docker rmi -f $(docker images -a --filter=dangling=true -q)`: remove unused images

#### Tips

Before you run `docker start deploy` you need to run `docker swarm init` first, otherwise you will get
error message *this node is not a swarm manager*.

- [Docker for beginner](https://github.com/docker/labs/blob/master/beginner/chapters/alpine.md)

Best practices:
- https://hackernoon.com/tips-to-reduce-docker-image-sizes-876095da3b34

