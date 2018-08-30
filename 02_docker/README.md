# Docker
This is the documentation for an **Introduction to Docker** for Redapt's "Docker Workshop".

by Jerry Meisner and Christoph Champ for Redapt, Inc. (August 2018)

**Docker** is an open-source project that automates the deployment of applications inside software containers.
> Docker containers wrap up a piece of software in a complete filesystem that contains everything it needs to run: code, runtime, system tools, system libraries – anything you can install on a server. This guarantees that it will always run the same, regardless of the environment it is running in. [Docker.com](https://www.docker.com/what-docker)

*NOTE: This document assumes you already have Docker installed. If you do not already have Docker installed, please use the directions found [here](INSTALL.md) (note: only for Linux).*

## Table of Contents
1. [Overview](01_overview/README.md)
   1. [Benefits of Containers](01_overview/01_benefits_of_containers.md)
   2. [Install Docker](01_overview/02_install_docker.md)
   3. [Docker commands](01_overview/03_docker_commands.md)
   4. [Docker Directives](01_overview/04_docker_directives.md)
   5. [Container Volume Management](01_overview/05_container_volume_management.md)
   6. [Images](01_overview/06_images.md)
   7. [Networking](01_overview/07_networking.md)
   8. [Monitoring](01_overview/08_monitoring.md)
   9. [Events](01_overview/09_events.md)
   10. Docker Image Repositories (Public/Private)
2. [Docker Compose](02_docker_compose)
3. [Containerization](03_containerization)
    1. [Dependencies and Image Size](03_containerization/01_dependencies_and_image_size.md)
    2. [Configurability and Security](03_containerization/02_configurability_and_security.md)
    3. [Tagging and Versioning](03_containerization/03_tagging_versioning.md)
    4. [.NET Core - ReactJS](03_containerization/04_netcore_react.md)
    5. [Java Spring Boot](03_containerization/05_java_spring.md)
    6. [Windows Containers](03_containerization/06_windows.md)