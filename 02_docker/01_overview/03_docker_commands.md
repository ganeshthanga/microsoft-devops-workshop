# Docker Commands

*Note: We will provide detailed examples on many of the following commands throughout this workshop.*

## Basics

The following are the most common Docker commands (i.e., the ones you will most likely use the most day-to-day):

* Show all running containers:
```
$ docker ps
```
* Show all containers (including stopped and failed ones):
```
$ docker ps -a
```
* Show all images in your local repository:
```
$ docker images
```
* Create an image based on the instructions in a <code>Dockerfile</code>:
```
$ docker build
```
* Start a container from an image (either from your local repository or from a remote repository {e.g., Docker Hub}):
```
$ docker run
```
* Remove/delete all ''stopped''/''failed'' containers (leaves running containers alone):
```
$ docker rm $(docker ps -a -q)
```

## Container commands

### Container lifecycle

* Create a container but do not start it:
```
$ docker create
```
* Rename a container:
```
$ docker rename
```
* Create *and* start a container in one operation:
```
$ docker run
```
* Delete a container:
```
$ docker rm
```
* Update a container's resource limits:
```
$ docker update
```

### Starting and stopping containers

* Start a container:
```
$ docker start
```
* Stop a running container:
```
$ docker stop
```
* Stop and start start a container:
```
$ docker restart
```
* Pause a running container ("freeze" it in place):
```
$ docker pause
```
* Un-pause a paused container:
```
$ docker unpause
```
* Attach/connect to a running container:
```
$ docker attach
```
* Block until running container stops (and print exit code):
```
$ docker wait
```
* Send <code>SIGKILL</code> to a running container:
```
$ docker kill
```

### Information

* Show all *running* containers:
```
$ docker ps
```
* Get the logs for a given container:
```
$ docker logs
```
* Get all of the metadata about a container (e.g., IP address, etc.):
```
$ docker inspect
```
* Get real-time events from Docker Engine (e.g., start/stop containers, attach, create, etc.):
```
$ docker events
```
* Get the public-facing ports of a given container:
```
$ docker port
```
* Show running processes in a given container:
```
$ docker top
```
* Show a given container's resource usage statistics:
```
$ docker stats
```
* Show changed files in the container's filesystem (i.e., those changed from the original base image):
```
$ docker diff
```

## Image commands

### Lifecycle

* Show all images in your local repository:
```
$ docker images
```
* Create an image from a tarball:
```
$ docker import
```
* Create an image from a <code>Dockerfile</code>
```
$ docker build
```
* Create an image from a container (note: it will pause the container, if it is running, during the commit process):
```
$ docker commit
```
* Remove/delete an image:
```
$ docker rmi
```
* Load an image from a tarball as STDIN (including images and tags):
```
$ docker load
```
* Save an image to a tarball (streamed to STDOUT with all parents lays, tags, and versions):
```
$ docker save
```

### Info

* Show the history of an image:
```
$ docker history
```
* Tag an image:
```
$ docker tag
```

### Miscellaneous

* Get the environment variables for a given container:
```
$ docker run ubuntu env
```
* IP address of host machine:
```
$ ip -4 -o addr show eth0
2: eth0    inet 10.0.0.166/23
```
* IP address of a container:
```
$ docker run ubuntu ip -4 -o addr show eth0
2: eth0    inet 172.17.0.2/16
```
