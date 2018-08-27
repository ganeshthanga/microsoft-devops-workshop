# Docker
This is the documentation for an **Introduction to Docker** for Redapt's "Docker Workshop".

by Jerry Meisner and Christoph Champ for Redapt, Inc. (August 2018)

**Docker** is an open-source project that automates the deployment of applications inside software containers.
> Docker containers wrap up a piece of software in a complete filesystem that contains everything it needs to run: code, runtime, system tools, system libraries – anything you can install on a server. This guarantees that it will always run the same, regardless of the environment it is running in. [Docker.com](https://www.docker.com/what-docker)

*NOTE: This document assumes you already have Docker installed. If you do not already have Docker installed, please use the directions found [here](INSTALL.md) (note: only for Linux).*

## Table of Contents
1. [Overview](01_overview/README.md)
   1. [Benefits of Containers](01_overview/01_benefits_of_containers.md)
   2. [Docker commands](01_overview/02_docker_commands.md)
2. [Docker Directives](02_docker_directives/README.md)
3. [Container Volume Management](03_container_volume_management/README.md)
4. [Images](04_images/README.md)
5. [Networking](05_networking/README.md)
6. [Monitoring](06_monitoring/README.md)
7. [Events](07_events/README.md)
8. [Docker Compose](08_docker_compose/README.md)
9. Containerization
  1. Java [JVM Specifics]
  2. .NET (Windows Base Containers)
10. Configurability
11. Security