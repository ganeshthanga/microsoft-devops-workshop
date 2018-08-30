# Windows

Containerizing applications that are designed to run on Windows requires an understanding of the windows container landscape. 

There are two primary windows base images:

1. `microsoft/windowsservercore` [.NET Framework] - Larger image, compatible with existing Windows apps, IIS 
2. `microsoft/nanoserver` [.NET Core] - Much smaller image, used to deploy applications that run in the background and don't use the services of a full server OS, and may even laucn their own web server such as Kestrel. Microsoft has published a scanner that can determine if your app can be run on nanoserver. https://blogs.technet.microsoft.com/nanoserver/2016/04/27/nanoserverapiscan-exe-updated-for-tp5/

There are a few Microsoft supported variants of these 2 base images, listed at the bottom of this page.
https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/net-core-net-framework-containers/net-container-os-targets

You can see the base image we used for the netcore react demo in section 1: `microsoft/dotnet:2.1-aspnetcore-runtime`

Building from this particular base image, the resulting container is larger, but has the added benefit of being able to be run on both Linux and Windows host operating systems.

The most important thing to understand is that `windowsservercore` containers must be run on a Windows host machine. Docker must also be explicitly setup on that machine to run Windows containers. There are experimental features that you can enable to support both Linux and Windows containers, though typically the node is designated to run one or the other.