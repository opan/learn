# Docker

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

#### Docker Concepts

The use of linux containers to deploy applications is called __containerizations__

#### Docker terminology

- __Images__: The blueprints of our application which form the basis of containers. We used `docker pull` command to download the image.
- __Containers__: Created from Docker image and run the actual application. We create container using `docker run` with image we download with `docker pull` command. A list of running containers can be seen using the `docker ps` command.
- __Docker Daemon__: The background service running on the host that manages building, running and distributing Docker containers.
  The daemon is the process that runs in the operating system to which clients talk to.
- __Docker Client__: The command line tool that allows the user to interact with the daemon.
- __Docker Hub__: A registry of docker images. You can think of the registry as a directory of all available docker images.

#### Most used commands in Docket

- `docker --version`: check docker version.
- `docker pull`: download docker image.
- `docker run`: create new container. To test is docker installation is complete you can run `docker run hello-world`.
  You may often use this command and combine it with flags:
  - `docker run -d -p 80:80 --name webserver package-name`:
    - `-d`/`--detach` flag make you run container in background and print container ID.
    - `-p`/`--publish` flag to publish your container with specific port(s).
    - `--name` flag to give your container a name.

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

#### Tips

Before you run `docker start deploy` you need to run `docker swarm init` first, otherwise you will get
error message *this node is not a swarm manager*.
