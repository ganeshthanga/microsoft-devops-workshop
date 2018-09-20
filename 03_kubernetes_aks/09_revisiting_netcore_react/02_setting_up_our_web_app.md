# Setting Up Our Web App (Favorite Beer)

The second microservice in our stack is a stateless service. We want to be able to scale this independently, in other words launch as many pods as necessary, to handle our load. The best Kubernetes primitive to choose for this scenario is a `Deployment`, which will manage `ReplicationController`s for each rollout, which will manage the number of pods we have running.

We can map our understanding of the `docker-compose.yml` we developed in our local environment, to our understanding of this primitive in Kubernetes.

```
$ cat docker-compose.alt.yml

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
```
$ cat favorite-beer-deployment.yml

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
5. We have defined `REDIS_HOST` as `favorite-beer-redis`, as this is the `Service` name for our redis deployment, in the last section.
6. `ports` becomes more defined, since we prefer to name ports in Kubernetes, and we have more options available.

## Configuration

That implementation is great, if we want to use one of the servicesettings.json files that is packaged within the application. However, we exposed these things to be able to update our settings at deploy time, so we can create a deployment that uses configmaps and volume mounts with custom configuration.

Consider we define a configmap that includes a `servicesettings.json` with a few additional beverages, that looks like this:

```
$ cat favorite-beer-configmap.yml

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

$ kubectl apply -f favorite-beer-configmap.yml
```

We can then add to and reference this in our previous deployment:
```
$ cat favorite-beer-deployment.yml

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

$ kubectl apply -f favorite-beer-deployment.yml
```

What is happening here:
1. We define a volume in the volumes section called `favorite-beer-config-volume` which is referencing our `configMap` by name `favorite-beer`.
2. We are creating a mount to the container in the `volumeMounts` section, referencing the volume we defined, and the path within the container we want to mount to.
3. After these steps are performed, our container should have files that relate back to the confimap data keys, with the contents being their value. In this case `ls /etc/config/` would show `servicesettings.json`, so we update the `SERVICE_SETTINGS_PATH` environment variable pointing to this location.

Later when this app is deployed to Kubernetes, we will deploy `favorite-beer-configmap.yml` before or at the same time as `favorite-beer-deployment.yml`.

## Exposing this Deployment

Since this is a service that recieve traffic internally or otherwise over the net, we will need to expose an endpoint. This is where the Kubernetes `Service` comes into play. As you may remember, when we were discussing the primitives there are 3 types of services each more complex than the last, building up to `LoadBalancer`.

We can define a service for our deployment. If you already have a cluster running in AKS, you can speciy `type: LoadBalancer`, however on minikube you should use `NodePort`. This process executes iptables rules on every node, in the cluster for the NodePort defined. If not defined, it will choose a random available port, within a specific range. 

On AKS, with `LoadBalancer` this will additionally configure an Azure Load Balancer attached to every node in your cluster, with traffic routed to this NodePort. You could then setup your DNS servers to point to that Load Balancer, for a complete setup.

```
$ cat favorite-beer-service.yml

apiVersion: v1
kind: Service
metadata:
  name: favorite-beer
spec:
  selector:
    app: favorite-beer
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 31650
  type: LoadBalancer

$ kubectl apply -f favorite-beer-service.yml

$ kubectl get svc
NAME                  TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
favorite-beer         LoadBalancer   10.0.47.83     13.64.173.174   80:31650/TCP   26m
```

Note the LoadBalancer IP (13.64.173.174, in this example) Azure assigned to the Azure Load Balancer it attached to your AKS nodes. Put whichever IP Azure assigned for _your_ Load Balancer into your browser and you should see the "Favorite Beer" app working.
