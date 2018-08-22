# Docker directives

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build`, users can create an automated build that executes several command-line instructions in succession.

This section will describe some of the common Dockerfile commands (aka "instructions" / "directives").

### FROM

The `FROM` instruction initializes a new build stage and sets the Base Image for subsequent instructions. As such, a valid Dockerfile must start with a `FROM` instruction. The image can be any valid image – it is especially easy to start by pulling an image from the [Public Repositories](https://hub.docker.com/). All of the examples shown in this workshop will pull base images from the public repositories (aka Docker Hub).

Every example of a Dockerfile shown in this workshop will begin with the `FROM` directive.

### RUN

The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.

Layering `RUN` instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image’s history, much like source control.

Notes on the order of execution:
```
FROM centos:latest
LABEL maintainer="bob@example.com"

RUN useradd -ms /bin/bash bob
USER bob

RUN echo "export PATH=/path/to/my/app:$PATH" >> /etc/bashrc
```

```
$ docker build -t centos7/config:v1 .
 ...
 /bin/sh: /etc/bashrc: Permission denied
```

The order of execution matters! Prior to the directive `USER bob`, the user was root. After that directive, the user is now bob, who does not have super-user privileges. Move the `RUN echo ...` directive to before the `USER bob` directive for a successful build.

### USER

The `USER` instruction sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD`, and `ENTRYPOINT` instructions that follow it in the Dockerfile.

```
$ cat << EOF > Dockerfile
# Non-privileged user entry
FROM centos:latest
MAINTAINER bob@example.com

RUN useradd -ms /bin/bash bob
USER bob
EOF
```

*Note: The use of `MAINTAINER` has been deprecated in newer versions of Docker. You should use `LABEL` instead, as it is much more flexible and its key/values show up in `docker inspect`. From here forward, I will only use `LABEL`.*

```
$ docker build -t centos7/nonroot:v1 .
$ docker exec -it <container_name> /bin/bash
```

We are user "bob" and are unable to become root. The workaround (i.e., how to become root) is like so:

```
$ docker exec -u 0 -it <container_name> /bin/bash
```

*NOTE: For the remainder of this section, I will omit the `$ cat << EOF > Dockerfile` part in the examples for brevity.*

### ENV

The `ENV` instruction sets the environment variable `key` to the value `value`. This value will be in the environment of all "descendant" Dockerfile commands and can be replaced inline in many as well.

*Note: The following is a **_terrible_** way of building a container. I am purposely doing it this way so I can show you a much better way later (see below).*

* Build a CentOS 7 Docker image with Oracle Java 8 installed:
```
# SEE: https://gist.github.com/P7h/9741922 for various Java versions
FROM centos:latest
LABEL maintainer="bob@example.com"

RUN yum update -y
RUN yum install -y net-tools wget

RUN echo "SETTING UP JAVA"
# The tarball method:
# RUN cd ~ && wget --no-cookies --no-check-certificate \
#     --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
#     "http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.tar.gz"
# RUN tar xzvf jdk-8u91-linux-x64.tar.gz
# RUN mv jdk1.8.0_91 /opt
# ENV JAVA_HOME /opt/jdk1.8.0_91/

# The rpm method:
RUN cd ~ && wget --no-cookies --no-check-certificate \
    --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
    "http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.rpm"
RUN yum localinstall -y /root/jdk-8u161-linux-x64.rpm

RUN useradd -ms /bin/bash bob
USER bob

# User specific environment variable
RUN cd ~ && echo "export JAVA_HOME=/usr/java/jdk1.8.0_161/jre" >> ~/.bashrc
# Global (system-wide) environment variable
ENV JAVA_BIN /usr/java/jdk1.8.0_161/jre/bin
```

```
$ docker build -t centos7/java8:v1 .
```

### CMD vs. RUN

The main purpose of a `CMD` is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an `ENTRYPOINT` instruction as well.

There can only be one `CMD` instruction in a Dockerfile. If you list more than one `CMD` then only the last `CMD` will take effect.

```
FROM centos:latest
LABEL maintainer="bob@example.com"

RUN useradd -ms /bin/bash bob
CMD ["echo", "Hello from within my container"]
```

The `CMD` directive *only* executes when the container is started, whereas the `RUN` directive is executed during the build of the image.

```
$ docker build -t centos7/echo:v1 .
$ docker run centos7/echo:v1
Hello from within my container
```

The container starts, echos out that message, then exits.

### ENTRYPOINT

An `ENTRYPOINT` allows you to configure a container that will run as an executable. Examples include an Apache or Nginx webserver, a Python Jupyter notebook, an API service, etc.

```
FROM centos:latest
LABEL maintainer="bob@example.com"

RUN useradd -ms /bin/bash bob
ENTRYPOINT "This command will display this message on EVERY container that is run from it"
```

```
$ docker build -t centos7/entry:v1 .

$ docker run centos7/entry:v1
This command will display this message on EVERY container that is run from it

$ docker run centos7/entry:v1 /bin/echo "Can you see me?"
This command will display this message on EVERY container that is run from it

$ docker run centos7/echo:v1 /bin/echo "Can you see me?"
Can you see me?
```
Note the difference between the output of the "echo" and the "entry" containers.

### EXPOSE

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

The `EXPOSE` instruction does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. To actually publish the port when running the container, use the `-p` flag on docker run to publish and map one or more ports, or the `-P` flag to publish all exposed ports and map them to high-order ports (i.e., from 32769 - 65535).

```
FROM centos:latest
LABEL maintainer="bob@example.com"

RUN yum update -y
RUN yum install -y httpd net-tools

RUN echo "This is a custom index file built during the image creation" > /var/www/html/index.html

ENTRYPOINT apachectl -DFOREGROUND  # BAD WAY TO DO THIS!
```

```
$ docker build -t centos7/apache:v1 .

# Run the above built image as a container in dettached mode:
$ docker run -d --name webserver centos7/apache:v1

$ docker exec webserver /bin/cat /var/www/html/index.html
This is a custom index file built during the image creation

# Get the internal container IP address:
$ docker inspect webserver -f '{{.NetworkSettings.IPAddress}}'  # => 172.17.0.6
#~OR~
$ docker inspect webserver | jq -crM '.[] | .NetworkSettings.IPAddress'  # => 172.17.0.6

$ curl 172.17.0.6
This is a custom index file built during the image creation
$ curl -sI 172.17.0.6 | awk '/^HTTP|^Server/{print}'
HTTP/1.1 200 OK
Server: Apache/2.4.6 (CentOS)

# Stop the container:
$ time docker stop webserver
real   0m10.275s  # <- notice how long it took to stop the container
user   0m0.008s
sys    0m0.000s

# Remove the container
$ docker rm webserver
```

It took ~10 seconds to stop the above container. This is because of the way we are (incorrectly) using `ENTRYPOINT`. The `SIGTERM` signal when running `docker stop webserver` actually timed out instead of exiting gracefully. A much better method is shown below, which *will* exit gracefully and in less than 300 ms.

* Expose ports from the CLI
```
$ docker run -d --name webserver -p 8080:80 centos7/apache:v1
$ curl localhost:8080
This is a custom index file built during the image creation
$ docker stop webserver && docker rm webserver
```

* Explicitly expose a port in the Docker image:
```
FROM centos:latest
LABEL maintainer="bob@example.com"

RUN yum update -y && \
    yum install -y httpd net-tools && \
    yum autoremove -y && \
    echo "This is a custom index file built during the image creation" > /var/www/html/index.html

EXPOSE 80

ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

```
$ docker build -t centos7/apache:v2 .
$ docker run -d --rm --name webserver -P centos7/apache:v1

# Get the higher-order external port automatically assigned to the container:
$ docker container ls --format '{{.Names}} {{.Ports}}'
webserver 0.0.0.0:32769->80/tcp
#~OR~
$ docker port webserver | cut -d: -f2
32769
#~OR~
$ docker inspect webserver | jq -crM '[.[] | .NetworkSettings.Ports."80/tcp"[] | .HostPort] | .[]'
32769

$ curl localhost:32769
This is a custom index file built during the image creation

# Stop the container
$ time docker stop webserver
real   0m0.283s  # <- Note how much faster the container stopped
user   0m0.004s
sys    0m0.008s
```

Note that we passed `--rm` to the `docker run` command so that the container will be removed when we stop the container. Also note how much faster the container stopped (~300ms vs. 10 seconds above).
