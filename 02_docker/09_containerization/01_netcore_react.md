# .NET Core - ReactJS

For the following demo, we will be working from the following directory:

cd `examples/netcore-react/spa-react-netcore-redis/voting/voting`

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

We have configured a docker-compose.yml and a docker-compose.alt.yml to showcase the features. You saw in a previous section, how the docker compose file maps to docker commands.

We can build our image with docker compose, using the following command:

`docker-compose -f ../docker-compose.alt.yml build`

## Running the Container Locally

This application is engineered to respond on port 80, so when we run it, we want to specify that as well. The following command will run our container, detach from the process, and mount 8000 on our host to port 80 on the `voting` container we build above. 

We have also set up our application code, to pull redis host information from environment variables, so we will specify those, to point to our linked redis container. 

`SERVICE_SETTINGS_PATH` is an environment variable that is used to specify the path to our service settings json file. We build two of these into the image, `servicesettings.json` and `servicesettings.alt.json`. When we deploy to container management system, we would mount our json file to a path on the container, and point the application to that file, via this environment variable. If not passed in, we have chosen to default to `servicesettings.json`.

To enable persistence for redis, we have to override the command, and specify a local host volume. This is important information to know and test locally, prior to building out Kubernetes components for this application. We will discuss this further in later sections.

**docker-compose.alt.yml**
```
version: '3.4'

services:
  voting:
    build:
      context: .
      dockerfile: voting/Dockerfile
    depends_on:
      - redis
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - SERVICE_SETTINGS_PATH=servicesettings_alt.json
    ports: 
      - '8000:80'
  redis:
    image: "redis:3.2-alpine"
    command: ["redis-server", "--appendonly", "yes"]
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data
volumes:
  redis_data:
    driver: "local"

```

`docker-compose -f ../docker-compose.alt.yml up`

You may be able to hit [localhost:8000](http://localhost:8000), if you cannot try `docker ps` and then `docker inspect <name>` to get the `IPAddress`. If your containers are running in a VM, you may need to use that VM's ip, and port `8000`.

## Cleanup
In this example, we are using a local docker volume, to persist our data, so if you want to completely remove the data, you can run the following commands.

```
docker rm voting_redis_1
docker volume rm voting_redis_data
```

If you see issues, you can run `docker volume list` to see the volumes that are present.

