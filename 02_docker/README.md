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
9. Docker Image Repositories (Public/Private)
10. [Containerization](10_containerization)
    1. [Dependencies and Image Size](10_containerization/01_dependencies_and_image_size.md)
    2. [Configurability and Security](10_containerization/02_configurability_and_security.md)
    3. [Tagging and Versioning](10_containerization/03_tagging_versioning.md)
    4. [.NET Core - ReactJS](10_containerization/04_netcore_react.md)
    5. [Java Spring Boot](10_containerization/05_java_spring.md)
    6. [Windows Containers](10_containerization/06_windows.md)