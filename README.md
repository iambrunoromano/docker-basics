# Docker Basics

This single-file repo will briefly go over:
1. [Docker Theory](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#docker-theory)
2. [Run Container](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#run-container):
    - Ubuntu
    - NGINX
3. [Containers management](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#containers-management)
4. [Images management](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#images-management)
5. [Volume on NGNIX container](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#volume-on-ngnix-container)
6. [Dockerfile](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#dockerfile)
    - NGINX
7. [Docker Compose](https://github.com/iambrunoromano/docker-basics/blob/master/README.md#docker-compose)
    - Dockerfile for Compose application
    - Docker Compose file
    - Running Docker Compose infrastructure
    - Compose bind mount
    - Compose options

## Docker Theory

[Docker Engine](https://docs.docker.com/get-started/overview/) is a *client-server* application made by the following components:
- `server` daemon: long running program
- `bridge` REST API: communication bridge between `server` and `client`
- `client` docker CLI

<details>
  <summary>Click to know more about docker theory</summary>

The docker daemon manages docker objects like:
1. `images`: read-only templates with instructions to create containers
2. `containers`: runnable instances of images, are *startable*, *stoppable*, *movable* and *deletable*
3. `storage`: spaces to store files between containers, can be *volumes*, *bind mounts* and *tmpfs mounts*
4. `network`: the networking functionality includes drivers like the *bridge* (default driver when running application with standalone containers on same host), *host* (the best to not isolate network stack from host, but isolating other aspects of containers), *overlay* (best when you have several containers running on different docker hosts) or *macvlan* (best for migrating from VM setup, because associates a MAC for each container)

</details>

## Run Container

### Ubuntu

```
docker run -it ubuntu
```

<details>
  <summary>Click to know more about how to run an Ubuntu container</summary>

  The two options:
  1. `-t` allocates a [pseudo-TTY](https://docs.docker.com/reference/cli/docker/container/run/) (which means "teletype" and is a device that offers basic input-output): therefore the option allows the user to provide commands (as *sudo*) and read output from inside the container as a terminal console
  2. `-i`: `docker run` allows only output to be printed without a way to send input to the container so `-i` is used to attach the STDIN from the host terminal to the main process inside the container

  For more please refer to [Baeldung Linux guide](https://www.baeldung.com/linux/docker-run-interactive-tty-options).

  To exit from the Ubuntu shell type `CTRL + p + q` to leave the container running in the background. To get back into the container shell use `docker attach {container-id}`. The latter command executed the `COMMAND` of the specific container, but it's not the only option to get back into container console. It works fine when the `COMMAND` for the container is `/bin/bash`.

</details>

### NGINX

The command for running `nginx` container should need `-it` for running:
```
docker run -it nginx
```

<details>
  <summary>Click to know more about how to run an NGINX container</summary>

It doesn't really `-it` need it because not accepting any input from console, so we can use the detached mode to run this container:
```
docker run -d nginx
```
that runs the `nginx` container as daemon.
Instead, if you want to bind the internal container port to the external container port run:
```
docker run -d -p 8080:80 nginx
```

</details>

## Containers management

To show all containers, even the stopped ones:
```
docker ps -a
```

<details>
  <summary>Click to know more about containers management</summary>

To start an existing container given the `container-id`:
```
docker start {container-id}
```
To attach and get into a container given the `container-id`:
```
docker attach {container-id}
```
To leave the container shell type `CTRL + p + q`. 

To stop a container given the `container-id`:
```
docker stop {container-id}
```
To kill and forcefully exit a container given the `container-id`:
```
docker kill {container-id}
```
To remove a container given the `container-id`:
```
docker rm {container-id}
```
To show all the containers but only by `container-id`:
```
docker ps -aq
```
To remove all the containers:
```
docker rm $(docker ps -aq)
```
To forcefully remove a running container given the `container-id`
```
docker rm -f {container-id}
```

</details>

## Images management

To show all images:
```
docker images -a
```

<details>
  <summary>Click to know more about images management</summary>

To remove images by `image-id`:
```
docker rmi {image-id}
```
To remove all images:
```
docker rmi $(docker images -qa)
```
To pull some latest docker image from registry:
```
docker pull image-name
```
`docker run image-name` includes the `docker pull image-name` if image is locally not existing.
To login with Docker ID to Docker Hub use:
```
docker login
```
and insert username and password. In this way you can pull and push your custom images from Docker Hub.

</details>

## Volume on NGNIX container

After running an `nginx` container create an `index.html` on the host. We need to create a volume (which is a mapping) from the folder of the `index.html` file to the container.

<details>
  <summary>Click to know more about volume on NGNIX container</summary>

To create the volume we need to:
1. Kill the previous container:
```
docker kill {container-id}
```
2. Run the new container using volume option:
```
docker run -d -p 8080:80 -v $(PWD)/nginx:/usr/share/nginx/html nginx
```
with `$(PWD)/nginx` the host folder in which `index.html` is located, `/usr/share/nginx/html` the container internal folder.

</details>

## Dockerfile

`Dockerfile` can contain all the commands that the user can call to assemble an image.

<details>
  <summary>Click to know more about Dockerfile</summary>
  
Using the `docker build` command one can create automated builds to execute different command-line instructions. `docker build .` builds the current directory.

Create a Dockerfile:
```
touch dockerfile 
```
Then insert in it:
```
FROM ubuntu:18.04
ENTRYPOINT
CMD
RUN touch /textfile.txt
```
If now you do the command
```
docker build . 
```
it starts building the image.
Running this image will run the ubuntu version described and create the file in the root folder:
```
docker run -it {image-id}
```
If you now do:
```
docker images 
```
you will see a new image with repository `<none>` and tag `<none>` that you wanted to create.
You can remove the image create with  
```
docker rmi {image-id}
```
If you want to create the image with a name and a tag use:
```
docker build -t repository-name:tag-name . 
```

</details>

### NGINX

To run a Dockerfile `nginx` server with interactive Linux CLI create a Dockerfile like the following:
```
FROM nginx
ENTRYPOINT /bin/bash
```

<details>
  <summary>Click to know more about NGINX Dockerfile</summary>
  
Then build the image as:
```
docker build -t test:v1
```
Now we have our image and running it with interactive CLI:
```
docker run -it {image-id}
```
If we want to add volumes to a Dockerfile you can do in the following way:
```
FROM ubuntu:18.04
ADD test1/ /home/test1
ADD test2/ /home/test2
```
Instead if you want to create a directory and go inside it you can use:
```
FROM ubuntu:18.04
WORKDIR /home/example_dir #WORKDIR = mkdir + cd commands
ADD test1/ ./
```

</details>

## Docker Compose
  
Tool to define and running multi-container applications. It's a three step process:
1. define a `Dockerfile` for the application environment
2. define the services to be included in the `docker-compose.yml` file that will be run together in isolated environment
3. run `docker compose up` to pull up all the containers

### Dockerfile for Compose application

Imagine you have an `app.py` Python application and a `requirements.txt` file with a bunch of requirements for our application in it:
```
flask
redis
```

<details>
  <summary>Click to know more about Dockerfile for Compose application</summary>
  
Let's create a docker file to execute python:
```
FROM python:3.7-alpine
WORKDIR /code # create and cd into /code folder
ENV FLASK_APP=app.py # create an environment variable as "app.py" for flask app name
ENV FLASK_RUN_HOST=0.0.0.0 # create an environment variable as "0.0.0.0" for flask host
RUN apk add --no-cache gcc musl-dev linux-header # requirements for python flask architecture
COPY requirements.txt requirements.txt # copying requirements.txt file from external to internal to the image
RUN pip install -r requirements.txt # make pip fetch and install all the requirements
EXPOSE 5000 # expose is  command that does nothing, it is like a comment, used to address the exposed ports
COPY . .
CMD ["flask","run"]
```

</details>

### Docker Compose file

Create a file named `docker-compose.yml` like the following:
```
version: "3.8"
services:
  web:
    build: . # this is like doing "docker build ."
    ports:
      - "5000:5000" # bind internal port 5000 to external port 5000
  redis:
    image: "redis:alpine"
```

### Running Docker Compose infrastructure
For running the whole infrastucture use:
```
docker-compose up -d
```

### Compose bind mount
If you want to update the application you can update the docker-compose.yml file as:
```
...
services:
  web:
    ...
    volumes:
      - .:/code # binding the external . to the internal /code directory
  ...
```
Adding volume binding you allow the web service update because in the volume we can always write a new list of requirements.

### Compose options
Use `docker-compose down` to take it down. Use `docker-compose run web env` to check the environment variables for the `web` service. 

### Credits
Credits to the [Docker Course](https://www.udemy.com/course/learn-docker-from-the-scratch-and-prepare-for-job-interview/?couponCode=LETSLEARNNOW) from [Musab AlZayadneh](https://www.udemy.com/user/musab-alzayadneh3)
