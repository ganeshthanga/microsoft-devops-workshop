# Containerization

When working with containers and developing your build strategy, there are a few things to consider. 

1. Dependencies
    - What Operating System does your application perform best on?
    - What packages do you need installed?
    - Does your app connect to external data-storage services?
2. Image Size
    - What is the smallest footprint for a functioning executable?
    - What build pattern should you choose?
3. Configurability
    - What configuration items might change between deployments?
4. Security
    - If your app connects to external services, how are credentials passed in?
5. Tagging / Versioning
    - Does the chosen tag make sense now and for future releases?

#### Development

It's important to get your development environment setup to work with containers, and understand the particulars when it comes to ip addressing, volumes, etc. For example, if you are running docker daemon in Virtual Box, you should understand that your containers are running on that VM, and networking settings at that level apply.

1. Ensure you can connect to the daemon by running a simple `docker ps`.
   - This has the added benefit of listing currently running containers, which you should remove, if possible.
2. Make sure you can run a simple container and expose a port, which you can hit via curl or browser request.
   - For example: `docker run -p 8000:80 nginx`


### Dependencies




### Examples

We have prepared a couple of examples, that can be built and deployed, during the workshop.

## Table of Contents
1. [.NET Core - ReactJS](01_netcore_react.md)
2. [Java Spring Boot](02_java_spring.md)
3. [Windows Containers](03_windows.md)