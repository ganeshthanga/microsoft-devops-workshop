# Revisiting .NET Core ReactJS

With some understanding of the primitives, we can take the application we built in the containerization section and deploy it to our cluster.

Our application stack includes two microservices. 
1. Our .NET Core app (The image has been publicly staged on docker hub at `redaptcloud/favorite-beer:v1`)
2. The redis data store.

The first microservice in our stack is a stateless service. We want to be able to scale this independently, in other words launch as many pods as necessary, to handle our load. The best Kubernetes primitive to choose for this scenario is a `Deployment`, which will manage `ReplicationController`s for each rollout, which will manage the number of pods we have running.

We can map our understanding of the docker-compose.yml we developed in our local environment, to our understanding of this primitive in Kubernetes.

docker-compose.alt.yml
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


Our deployment for the `voting`/`favorite-beer` app container.
`favorite-beer-deployment.yml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: favorite-beer
  labels:
    app: favorite-beer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: favorite-beer
  template:
    metadata:
      labels:
        app: favorite-beer
    spec:
      containers:
      - name: favorite-beer
        image: redaptcloud/favorite-beer:v1
        env:
        - name: REDIS_HOST
          value: "favorite-beer-redis"
        - name: REDIS_PORT
          value: "6379"
        - name: SERVICE_SETTINGS_PATH
          value: "servicesettings_alt.json"
        ports:
        - name: http
          containerPort: 80
```


Notable Differences:

1. The deployment metadata is included, and controller information are also included. 
2. The containers section is populated from the `voting` section, with `image` replacing `build`
3. `depends_on` is a docker-compose construct, that is excluded, but considered during deployment strategy. It tells us we will need to deploy our redis endpoint first, when we actually apply this to Kubernetes.
4. `environment` becomes `env` under the containers spec, and takes a different input format.
5. We have defined `REDIS_HOST` as `favorite-beer-redis`, because we will use a service primitive with the same name to expose redis to our deployment, when we deploy redis.
6. `ports` becomes more defined, since we prefer to name ports in Kubernetes, and we have more options available.


## Configuration

That implementation is great, if we want to use one of the servicesettings.json files that is packaged within the application. However, we exposed these things to be able to update our settings at deploy time, so we can create a deployment that uses configmaps and volume mounts with custom configuration.

Consider we define a configmap that includes a servicesettings.json with a few additional beverages, that looks like this:

`favorite-beer-configmap.yml`
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: favorite-beer
data:
  servicesettings.json: |
    {
      "ServiceSettings": {
        "Beers": ["Red Hook", "Pyramid", "Great Divide", "New Belgium"]
      }
    }
```

We can then add to and reference this in our previous deployment.
`favorite-beer-deployment.yml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: favorite-beer
  labels:
    app: favorite-beer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: favorite-beer
  template:
    metadata:
      labels:
        app: favorite-beer
    spec:
      containers:
      - name: favorite-beer
        image: redaptcloud/favorite-beer:v1
        env:
        - name: REDIS_HOST
          value: "favorite-beer-redis"
        - name: REDIS_PORT
          value: "6379"
        - name: SERVICE_SETTINGS_PATH
          value: "/etc/config/servicesettings.json"
        ports:
        - name: http
          containerPort: 80
        volumeMounts:
        - name: favorite-beer-config-volume
          mountPath: /etc/config
      volumes:
        - name: favorite-beer-config-volume
          configMap:
            name: favorite-beer
```

What is happening here:
1. We define a volume in the volumes section called `favorite-beer-config-volume` which is referencing our `configMap` by name `favorite-beer`.
2. We are creating a mount to the container in the `volumeMounts` section, referencing the volume we defined, and the path within the container we want to mount to.
3. After these steps are performed, our container should have files that relate back to the confimap data keys, with the contents being their value. In this case `ls /etc/config/` would show `servicesettings.json`, so we update the `SERVICE_SETTINGS_PATH` environment variable pointing to this location.

Later when this app is deployed to Kubernetes, we will deploy `favorite-beer-configmap.yml` before or at the same time as `favorite-beer-deployment.yml`.

## Exposing this Deployment

Since this is a service that recieve traffic internally or otherwise over the net, we will need to expose an endpoint. This is where the Kubernetes `Service` comes into play. As you may remember, when we were discussing the primitives there are 3 types of services each more complex than the last, building up to `LoadBalancer`.

1. ClusterIP - This results in a local ip that can be reached by other pods within the cluster only.
2. NodePort - Includes a ClusterIP, but also creates an iptable rule on each node, at a designated port per service with this type, to allow external traffic.
3. LoadBalancer - Typically used with CloudProviders, this provides a ClusterIP, a NodePort, and additionally creates a cloud loadbalancer attached to your nodes on that NodePort.

We can define a service for our deployment. If you already have a cluster running in AKS, you can speciy `type: LoadBalancer`, however on minikube you should use `NodePort`. This process executes iptables rules on every node, in the cluster for the NodePort defined. If not defined, it will choose a random available port, within a specific range. 

On AKS, with `LoadBalancer` this will additionally configure an Azure Load Balancer attached to every node in your cluster, with traffic routed to this NodePort. You could then setup your DNS servers to point to that Load Balancer, for a complete setup.

`favorite-beer-service.yml`
```
apiVersion: v1
kind: Service
metadata:
  name: favorite-beer
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 31650
  type: NodePort
```

## Continuing to Define our Stack

In the next section, we discuss stateful apps, and will perform similar operations for redis.