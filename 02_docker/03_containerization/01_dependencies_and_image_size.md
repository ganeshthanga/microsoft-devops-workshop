# Dependencies and Image Size

There are some general questions that need to be answered before starting to build your first Dockerfile.

- Dependencies
    - What Operating System does your application perform best on?
    - What packages do you need installed?
- Image Size
    - What is the smallest footprint for a functioning executable?
    - What build pattern should you choose?

## Dependencies

There are a variety of public images available on Docker Hub (and other repositories), which have certain sets of dependencies already built in. Of course, there's a chance that these images have more in them than what you really need. You should always inspect the Dockerfile of any public repository and determine if it makes sense to grab a more "raw" base image and add commands to it. If the latter could produce a significantly smaller image, and remain functional, it's generally preferred.

## Image Size

You should aim for the smallest functional image size. When you are running your containers in an ochestrated environment, such as Kubernetes, each node may download your image, and that will consume some slice of the disk space. When you horizontally scale your pods, there is a chance the image will need to be download to a particular node. Reducing the size can improve the rate at which scale up can occur, since your container won't be running until the image is downloaded, and the process is started.

In most cases, this means compiling binaries outside of the final container, and bringing them in to a stripped down final container via copy. For Windows containers there is `nanoserver` and for Linux containers there is `alpine`, which both have the goal of being lightweight. Building your final image from these containers is recommended, if possible. (`nanoserver` only supports .NET Core applications, full framework apps need to use `windowsservercore`, see section 3 below.) 
