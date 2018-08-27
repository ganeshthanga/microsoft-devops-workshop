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

We have prepared a couple of examples, that can be built and deployed, during the workshop.

## Table of Contents
1. [.NET Core - ReactJS](01_netcore_react.md)
1. [Java Spring Boot](02_java_spring.md)