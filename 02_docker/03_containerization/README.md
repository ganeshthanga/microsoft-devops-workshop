# Containerization

This section is dedicated to working through building a docker image, such as questions to ask, and how to provide the answers via Dockerfile. The .NET Core - ReactJS application we will use to demonstrate Kubernetes and Jenkins, in later sections.

#### Development

It's important to get your development environment setup to work with containers, and understand the particulars when it comes to ip addressing, volumes, etc. For example, if you are running docker daemon in Virtual Box, you should understand that your containers are running on that VM, and networking settings at that level apply.

For the example, you should also have `docker-compose` installed and available via CLI.

1. Ensure you can connect to the daemon by running a simple `docker ps`.
   - This has the added benefit of listing currently running containers, which you should remove, if possible.
2. Make sure you can run a simple container and expose a port, which you can hit via curl or browser request.
   - For example: `docker run -p 8000:80 nginx`
3. Ensure that you have `docker-compose` installed and available via cli.
   - For example: `docker-compose` should print the manual.

### Examples and Information

We have prepared an example, that can be built and deployed, during the workshop.

## Table of Contents
1. [Dependencies and Image Size](01_dependencies_and_image_size.md)
2. [Configurability and Security](02_configurability_and_security.md)
3. [Tagging and Versioning](03_tagging_versioning.md)
4. [.NET Core - ReactJS](04_netcore_react.md)
5. [Java Spring Boot](05_java_spring.md)
6. [Windows Containers](06_windows.md)