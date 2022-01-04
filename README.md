# Docker Tutorial for ACM Cyber intern peeps

## What is Docker?

A software platform that allows one to test, build and deploy applications quickly through the use of containers. Think of containers like virtual machines (separate allocated resources for each machine and isolation), but without the overhead of the extra OS. In effect, containers are just applications bundled with their dependencies.

![container vs virtual machine](containers-vs-virtual-machines.jpg)

## Installation

Follow instructions [here](https://docs.docker.com/desktop/) to install docker desktop (which should come with docker cli and docker-compose).

Verify installation with the following:

```docker run hello-world```


## Learning Docker commands

First, pull this docker image from the main docker repo with the [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) command

```bash
docker pull vulnerables/web-dvwa
```

For reference, an image is what is used to create a container. It is an "ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime". This ordered collection of filesystem changes refers to how images are built through layers, which we will cover when we talk about Dockerfile. Note that we create our own images through Dockerfiles.

Now, we run our vulnerable application.

```bash
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```

You should be able to connect to the application on http://localhost or 192.168.99.100 for docker toolbox. Login with username admin, password admin. You can mess around with this application.

[Docker run](https://docs.docker.com/engine/reference/run/) simply runs a container based on the given image. See command format below.

```bash
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

Important options

[-p](https://docs.docker.com/engine/reference/run/#expose-incoming-ports) Publishes a container port to the host port. In above command, it sets localhost http port to use the containers http port

[-d](https://docs.docker.com/engine/reference/run/#detached--d) Run in detached mode (runs in background of terminal). So you can use your bash for something else.

[-it](https://docs.docker.com/engine/reference/run/#foreground) When running in the foreground, this is used to start interactive processes with a container (-i keeps STDIN open, -t allows a pseudo tty). This is used later when with docker exec

[--rm](https://docs.docker.com/engine/reference/run/#clean-up---rm) Cleans up leftover file system after running a container. (includes volumes)

Stop the docker run with ctrl-c. Launch it in detached mode.

Now, check on our local images with

```bash
docker images
```
This should have some list of images including hello-world, php and vulnerables/web-dvwa. Note the different columns-- TAG is just an associated tag that an image can have (like a version number) and IMAGE ID is just a unique id associated with each image. 

Let's check on our running containers

```bash
docker ps
```
If you currently launched DVWA in detached mode, it should show up here. 
You should see that it is running the command /main.sh.

If you run a container without --rm, you can also see it's remaining file system (even after the container stops running) if you add the -a option to this command.

Let's see whats in our running DVWA. Check the container id from the docker ps
```
docker exec -it [CONTAINER ID] /bin/bash
```

You should now be able to explore the file system of DVWA. Try looking around /var/www/html.

[Docker exec](https://docs.docker.com/engine/reference/commandline/exec/) is similar to docker run, but instead of creating a new container it runs in an existing container. See command format below.

```bash
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

Note that we can make it easier to docker exec into a container by naming our container on docker run with the --name options.

You can then kill your running containers with ```docker kill``` or ```docker stop```

## Learning Dockerfile

We're now going to build our own image. Create a file called Dockerfile in this folder.

Add the line `FROM php:7.4-cli` 

This defines the base image that we will be layering on top of. This particular image sets up a linux environment with everything php related configured. 

Add the line `COPY *.php /usr/src/myapp`

COPY and ADD simply add directories/files from the source in the current file system to the destination in the docker image. ADD is just COPY with added functionality; most of the time you should just use COPY.

Add the line `RUN touch test`

Simply executes the commands in the container. If in this form above, will execute the command in a command shell (/bin/sh -c by default) and in the form `RUN ["executable", "param1", "param2"]`, will not invoke a command shell.

Add the line `WORKDIR /usr/src/myapp`

Defines the working directory, from which subsequent RUN, CMD, ENTRYPOINT, COPY and ADD instructions will be run from. 

Add the line `CMD [ "php", "./home.php" ]`

CMD specifies which instruction is run when we launch up our container. For example, above we run our php server. This command can be overridden by the COMMAND argument to docker run.

ENTRYPOINT functions similarly, but it differs in the fact that while the last CMD overrides the CMDs before it, the ENTRYPOINT is always considered the main execution point. Subsequent CMDs act as arguments for the ENTRYPOINT. Thus, you can use ENTRYPOINT to house the command for a container that should have a specific executable, and then change the default parameter with CMD.

Now you should be able to use ```docker build``` to build this image

```
docker build .
```

Then check your ```docker images``` and you should be able to start the container with the image id.

Quick aside-- you can tag your images with the --tag flag

Other important commands

`ENV` just sets up environmental variables in the container

`USER` switches the user under which subsequent instructions are run-- like WORKDIR.

`EXPOSE` Adds metadata to the image for which port the container is listening on. Note that you still have to publish the port with -p.

`VOLUME` allows you to store stuff on the host file system instead of the containers, adding persistent data to the container. This directory isn't removed on docker rm, but it is removed on docker run --rm.

For best practice, it is recommended to minimize the number of layers, as the docker overhead adds up. This includes doing [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) and coalescing commands into single layers.

## Docker compose

Docker compose is a tool to set up and run multiple Docker containers without most of the hassle. It does this with an overarching docker-compose.yaml file in the root directory of a project, where the other subdirectories would have different projects with their own DockerFiles.

Check the docker-compose.yml file

Most of the "concepts" shown here should be familiar. Each services is its own container, and it can either be created from a pre-existing image or one that is built. 

Try launching up the docker containers with `docker-compose up -d `

You should see that this does all the work of `docker run` for you.

Docker-compose files can get alot more complex. Check the [documentation](https://docs.docker.com/compose/compose-file/) to learn some more about them.