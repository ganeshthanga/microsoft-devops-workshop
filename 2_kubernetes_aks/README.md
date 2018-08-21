# Kubernetes

Kubernetes (also known by its numeronym k8s) is an open source container cluster manager. Kubernetes' primary goal is to provide a platform for automating deployment, scaling, and operations of application containers across a cluster of hosts. Kubernetes was released by Google on July 2015.

Latest stable release: v1.11.1 (2018-08-17)

Get the latest stable release with:
```
$ curl -sSL https://dl.k8s.io/release/stable.txt
```

Kubernetes is built through the definition of a set of components (building blocks or "primitives") which, when used collectively, provide a method for the deployment, maintenance, and scalability of container-based application clusters.

These "primitives" are designed to be loosely coupled (i.e., where little to no knowledge of the other component definitions is needed to use) as well as easily extensible through an API. Both the internal components of Kubernetes as well as the extensions and containers make use of this API.

## Table of Contents

1. Overview / Moving Parts
   1. ETCD
   2. Kubernetes API
   3. Controller Manager / Scheduler / Kubelet / Proxy
   4. Networking (Flannel/Calico)
   5. Primitives (Deployments / Services / Pods / etc)
   6. RBAC / Service Accounts
2. Environment Strategies (NonProd/Prod vs Dev/Test/Cert/Prod)
   1. Namespacing
   2. Service Discovery / KubeDNS
   3. Workloads
3. Standing up your First Cluster
   1. MiniKube (Local)
   2. AKS
4. Deploying Stateless Applications
   1. Hello-World
   2. Config Maps / Secrets
5. Deploying Stateful Applications
   1. ELK Stack
   2. PVC
   3. Volume Types
6. System Services
   1. Standard Kube-System Pods
   2. Third Party Add-Ons (Datadog, Helm)
   3. Developing from Scratch
7. Ingress / Traffic Routing
   1. Defining Services ( ClusterIP / NodePort / LoadBalancer )
   2. Ingress Controllers / Rules
8. Auto-Scaling / SRE
   1. Resources (Limits / Requests)
   2. Horizontal Pod Autoscaling
   3. Cluster Scaling
   4. Liveness / Readiness Probes