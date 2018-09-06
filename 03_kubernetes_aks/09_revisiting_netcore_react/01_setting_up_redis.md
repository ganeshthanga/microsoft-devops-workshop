# Setting Up Redis

Since redis is a dependency of our app, we should deploy it first. The recommended way is via `StatefulSet`

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


Our deployment for the `voting`/`redis` container.
`favorite-beer-redis-statefulset.yml`
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: favorite-beer-redis
spec:
  serviceName: "favorite-beer-redis"
  replicas: 1
  selector:
    matchLabels:
      app: favorite-beer-redis
  template:
    metadata:
      labels:
        app: favorite-beer-redis
    spec:
      containers:
      - name: favorite-beer-redis
        image: redaptcloud/redis:3.2-alpine
        command: ["redis-server"]
        args: ["--appendonly", "yes"]
        ports:
        - containerPort: 6379
          name: redis
```


Notable Differences:
1. The deployment metadata is included, and controller information are also included. 
2. The containers section is populated from the `redis` section, and command is split into command and args.
3. `ports` becomes more defined, since we prefer to name ports in Kubernetes, and we have more options available.
4. We have not included any information about volumes, getting moutned to the container, yet.

## Persistent Volumes

While this implementation could be used, if we are concerned about keeping this data, for the life of our cluster, we will need to create a PersistentVolume.

#### MiniKube

**NOTE** Local storage is not compatible with Dynamic Provisioning of volumes, so we take a different approach for minikube.

On minikube, we can experiment with this by creating a host path volume, that we will use for our volume claim.

`minkube ssh`

`mkdir /data/redis-data`

favorite-beer-redis-pv.yml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: favorite-beer-redis-pv
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  capacity:
    storage: 1Gi
  hostPath:
    path: /data/redis-data/
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: favorite-beer-redis-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

`kubectl apply -f favorite-beer-redis-pv.yml`

favorite-beer-redis-statefulset.yml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: favorite-beer-redis
spec:
  serviceName: "favorite-beer-redis"
  replicas: 1
  selector:
    matchLabels:
      app: favorite-beer-redis
  template:
    metadata:
      labels:
        app: favorite-beer-redis
    spec:
      containers:
      - name: favorite-beer-redis
        image: redaptcloud/redis:v1
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumes:
    - name: redis-date
      persistentVolumeClaim:
       claimName: favorite-beer-redis-pvc
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - favorite-beer-redis
```

`kubectl apply -f favorite-beer-redis-statefulset.yml`

Notable Differences:

1. A PersistentVolume and a PVC were created, specifying a hostPath that we want to mount our data to on our minikube node. The claim will consume the volume, when it is created.
2. In our set, we reference the claim in the volumes section, and then specify the path in our container to mount it to, in the volumeMounts section.
2. Anti Affinity - Added to prevent collisions with local hostpaths for minikube users, but for AKS, it can help satisfy availability, requirements if the pods are not colocated. The caveat being, you will need to have at least as many nodes, as pods you desire in the stateful set. In minikube, this is a single node cluster.

#### AKS

In AKS, we can store out data in a Premium Managed Disk really easily, with Dynamic Provisioning.

favorite-beer-redis-statefulset.yml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: favorite-beer-redis
spec:
  serviceName: "favorite-beer-redis"
  replicas: 1
  selector:
    matchLabels:
      app: favorite-beer-redis
  template:
    metadata:
      labels:
        app: favorite-beer-redis
    spec:
      containers:
      - name: favorite-beer-redis
        image: redaptcloud/redis:v1
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      storageClassName: managed-premium
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

`kubectl apply -f favorite-beer-redis-statefulset.yml`

Notable Differences:

1. In our set, we have specified a volumeClaimTemplate that will tell the AKS cloud provider, to create a Managed Disk and attach it to designated nodes for each pod that is created. We reference the claim template name and mount it to our containers data path, in the volumeMounts section.

## Exposing this StatefulSet

Since this is a service that recieve traffic internally or otherwise over the net, we will need to expose an endpoint. This is where the Kubernetes `Service` comes into play. As you may remember, when we were discussing the primitives there are 3 types of services each more complex than the last, building up to `LoadBalancer`.

We can define a service for our stateful set, and in this case we will choose `type: ClusterIP`. Applying this resource to our cluster, will create an internal (To the kubernetes cluster) ip that can be reached via the service name. We don't have any need to access our redis instance, outside of services running on Kubernetes, so no need to go further.

`favorite-beer-redis-service.yml`
```
apiVersion: v1
kind: Service
metadata:
  name: favorite-beer-redis
spec:
  selector:
    app: favorite-beer-redis
  ports:
  - name: redis
    port: 6379
    protocol: TCP
    targetPort: 6379
  type: ClusterIP
```

`kubectl apply -f favorite-beer-redis-service.yml`

## Continuing to Define our Stack

In the next section, we will detup the resources for our favorite-beer  web application.