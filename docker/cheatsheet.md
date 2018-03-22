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

#### Play with Docker busybox

- Download `busybox` image with `docker pull busybox`.
- 

#### Getting started

- Create new directory, lets call `sample`.
- `cd` into new folder then create file called `Dockerfile`.
- Copy and paste content below into `Dockerfile` as initial setup.
    ```
    # Use an official Python runtime as a parent image
    FROM python:2.7-slim

    # Set the working directory to /app
    WORKDIR /app

    # Copy the current directory contents into the container at /app
    ADD . /app

    # Install any needed packages specified in requirements.txt
    RUN pip install --trusted-host pypi.python.org -r requirements.txt

    # Make port 80 available to the world outside this container
    EXPOSE 80

    # Define environment variable
    ENV NAME World

    # Run app.py when the container launches
    CMD ["python", "app.py"]
    ```
- If you behind a proxy server, add code below into your `Dockerfile` before the call to `pip` command above
    ```
    # Set proxy server, replace host:port with values for your servers
    ENV http_proxy host:port
    ENV https_proxy host:port
    ```
- Create two new files that still don't exists, `requirements.txt` and `app.py` on same folder with `Dockerfile`.
  Then fill both files with:

  `requirements.txt`
    ```
    Flask
    Redis
    ```

  `app.py`
    ```python
    from flask import Flask
    from redis import Redis, RedisError
    import os
    import socket

    # Connect to Redis
    redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

    app = Flask(__name__)

    @app.route("/")
    def hello():
        try:
            visits = redis.incr("counter")
        except RedisError:
            visits = "<i>cannot connect to Redis, counter disabled</i>"

        html = "<h3>Hello {name}!</h3>" \
               "<b>Hostname:</b> {hostname}<br/>" \
               "<b>Visits:</b> {visits}"
        return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=80)
    ```
- Now run `docker build -t friendlyhello .` to build docker image. We are using `-t` flag to tag it with `friendlyhello`.
- Run the docker image with `docker run -p 4000:80 friendlyhello`

#### Getting started 2

- Make sure your docker installation ok with `docker run hello-world`. You will get output result something like this
    ```
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    ...
    ```


#### Most used commands in Docket

- `docker pull`: download docker image.
- `docker run`: create new container. To test is docker installation is complete you can run `docker run hello-world`.
- `docker info`: get details about your docker installation.
- `docker image ls`: list out the image you have downloaded (in this example `hello-world` image).
- `docker container ls --all`: list out `hello-world` container.
- `docker ps`: get list containers that are running. Add `-a` option to list all containers that we run
- `docker rm $(docker ps -a -q -f status=exited)`: you can use this to remove all containers that has been running. This command delete all containers that have status `status=exited`. `-q` flag to return only the numeric IDs and `-f` filters output based on conditions provided.

