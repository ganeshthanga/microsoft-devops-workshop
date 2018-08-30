# Containerization

When working with containers and developing your build strategy, there are a few things to consider. 

1. Dependencies
    - What Operating System does your application perform best on?
    - What packages do you need installed?
2. Image Size
    - What is the smallest footprint for a functioning executable?
    - What build pattern should you choose?
3. Configurability
    - What configuration items might change between deployments?
    - Does your app connect to external data-storage services?
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

There are a variety of public images available on Dockerhub, and other repositories, that have certain sets of dependencies already built in, of course theres a chance that these images have more in them, than what you really need. You should try to inspect the Dockerfile of any public repository, and determine if it makes sense to grab a more raw base image, and add commands to it. If the latter could produce a significantly smaller image, and remain functional, its generally preferred.

### Image Size

You should aim for the smallest functional image size. When you are running your containers in an ochestrated environment, such as Kubernetes, each node may download your image, and that will consume some slice of the disk space. When you horizontally scale your pods, there is a chance the image will need to be download to a particular node. Reducing the size can improve the rate at which scale up can occur, since your container wont be running until the image is downloaded, and the process is started.

In most cases this means compiling binaries outside of the final container, and bringing them in to a stripped down final container via copy. For Windows containers there is `nanoserver` and for Linux containers there is `alpine` which both have the goal of being lightweight. Building your final image from these containers is recommended, if possible. (`nanoserver` only supports .NET Core applications, full framework apps need to use `windowsservercore`, see section 3 below.) 

### Configurability

There are certain aspects of your application that may be external, such as a database or cache, and you will want a way to specify the connection credentials as env vars or configuration files. This may require a code change and a slight re-architecture, depending on your current build/deploy strategy.

### Security

There are at least a couple of layers to consider when talking container security. The first rule of thumb is to never bake secrets or credentials into your builds, or store them in source control, where the builds are built from. If you did this, and someone got a hold of your image, they could simply `exec` into it, and read your credentials.

The second priority should be on ensuring that packages and other plugins that you install, do not have their own security flaws. Tools like [Twistlock](https://www.twistlock.com/2016/11/07/azure-container-registry/) and [Aqua Security](https://blog.aquasec.com/image-vulnerability-scanning-in-azure-container-registry) support Azure container registry. There are also open source tools that integrate with most repositories like [Clair](https://github.com/coreos/clair).

### Tagging and Versioning

Generally, when pulling an image from a repositoy, if you do not specify an image tag, the docker client will download the image tagged `latest`. You can take advantage of this by pushing your latest build to this tag, but you should avoid pushing **only** to `latest`, as some systems will default to using whats stored locally, rather than checking for diffs and re-downloading the new latest.

When choosing a tag for your image, it should be something meaningful, that you can use to trace back to the source control commit. For instance, Jenkins build number could be used, because you can check that build in the Jenkins log, and determine which commit in SCM was used to build it. It's within reason to tag the images with the sha256 of the commit, in addition to other identifying information such as build number, etc. It really depends on what reverse lookup strategy you think would be best.

### Examples and Information

We have prepared an example, that can be built and deployed, during the workshop.

## Table of Contents
1. [.NET Core - ReactJS](01_netcore_react.md)
2. [Java Spring Boot](02_java_spring.md)
3. [Windows Containers](03_windows.md)