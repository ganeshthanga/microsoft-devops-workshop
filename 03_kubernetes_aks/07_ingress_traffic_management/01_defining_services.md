# Defining Services

Recall from the k8s primitives section, a Service is an abstraction on top of Pods, which provides a single IP address and DNS name by which the Pods can be accessed. This load balancing configuration is much easier to manage and helps scale Pods seamlessly.
Kubernetes can then provide service discovery and handle routing with the static IP for each pod as well as load balancing (round-robin based) connections to that service among the pods that match the label selector indicated.

There are 3 types of services each more complex than the last, building up to `LoadBalancer`.

1. __ClusterIP__ [Default] - This results in a local ip that can be reached by other pods within the cluster only, end result is round robin traffic delivery.
![ClusterIP](img/service_-_clusterip.png)
2. __NodePort__ - Includes a ClusterIP, but also creates an iptable rule on each node, at a designated port per service with this type, to allow external traffic.
![NodePort](img/service_-_nodeport.png)
3. __LoadBalancer__ - Typically used with CloudProviders, this provides a ClusterIP, a NodePort, and additionally creates a cloud loadbalancer attached to your nodes on that NodePort.
![LoadBalancer](img/service_-_loadbalancer.png)

## Examples

The following examples can be copied into local files, and applied via `kubectl apply -f <filename>`.

__Example Service #1__:

```
kind: Service
apiVersion: v1
metadata:
  name: my-service1
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

```
$ kubectl apply -f my-service1.yml
$ kubectl get svc my-service1 -o yaml
$ kubectl describe svc my-service1
```

__Example Service #2__:

```
kind: Service
apiVersion: v1
metadata:
  name: my-service2
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
    nodePort: 31560
  type: NodePort
```

__Example Service #3__: (Only works correctly with Cloud Provider K8s clusters such as AKS, not Minikube.)

```
kind: Service
apiVersion: v1
metadata:
  name: my-service3
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
    nodePort: 31560
  type: LoadBalancer
```

### Cleanup

```
$ kubectl get svc
$ kubectl delete svc <svc_name>
```

