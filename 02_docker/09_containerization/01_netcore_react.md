# .NET Core - ReactJS

For the following demo, we will be working from the following directory:

[examples/netcore-react/spa-react-netcore-redis/voting/voting](examples/netcore-react/spa-react-netcore-redis/voting/voting)

We have pre-developed an application that uses redis to track votes, for favorite beers. The back-end exposes an API that would allow a customer or bartender to enter a +1 for a particular beverage. Our UI component is a single-page ReactJS app. We want to serve the front-end and the back-end api from the same container, but plan to have redis running in a seperate container.

## Build Strategy

Taking a look at the Dockerfile, it can be broken into 3 parts:

**Base Runtime Image**
```
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
```

**Build with SDK**
```
FROM microsoft/dotnet:2.1-sdk AS build
# Install Node.js
RUN apt-get install --yes curl
RUN curl --silent  --location https://deb.nodesource.com/setup_8.x |  bash -
RUN apt-get install --yes nodejs
RUN apt-get install --yes build-essential
WORKDIR /src
COPY voting/voting.csproj voting/
RUN dotnet restore voting/voting.csproj
COPY . .
WORKDIR /src/voting
RUN dotnet build voting.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish voting.csproj -c Release -o /app
```

**Final Image Build with Artifacts**
```
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
EXPOSE 80
ENTRYPOINT ["dotnet", "voting.dll"]
```

You can see that we are using a compound build strategy, wherein the components necessary to package our application, will not be part of our deployable container. From the directory with the Dockerfile, the build can be kicked off like this:

`docker build -t voting -f Dockerfile ../`

This will take a few minutes.

## Running the Container Locally

This application is engineered to respond on port 80, so when we run it, we want to specify that as well. The following command will run our container, detach from the process, and mount 8000 on our host to port 80 on the `voting` container we build above. 

`docker run -d -p 8000:80 voting`

You may be able to hit [localhost:8000](http://localhost:8000), if you cannot try `docker ps` and then `docker inspect <name>` to get the `IPAddress`. If your containers are running in a VM, you may need to use that VM's ip, and port `8000`.


