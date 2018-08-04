# Kubernetes

Kubernetes (also known by its numeronym k8s) is an open source container cluster manager. Kubernetes' primary goal is to provide a platform for automating deployment, scaling, and operations of application containers across a cluster of hosts. Kubernetes was released by Google on July 2015.

Latest stable release: v1.11.1 (2018-08-17)

Get the latest stable release with:
```
$ curl -sSL https://dl.k8s.io/release/stable.txt
```

## Design Overview

Kubernetes is built through the definition of a set of components (building blocks or "primitives") which, when used collectively, provide a method for the deployment, maintenance, and scalability of container-based application clusters.

These "primitives" are designed to be loosely coupled (i.e., where little to no knowledge of the other component definitions is needed to use) as well as easily extensible through an API. Both the internal components of Kubernetes as well as the extensions and containers make use of this API.

## Components

The building blocks of Kubernetes are the following (note that these are also referred to as Kubernetes "Objects" or "API Primitives"):

<dl>
  <dt>Cluster</dt> 
  <dd>A cluster is a set of machines (physical or virtual) on which your applications are managed and run. All machines are managed as a cluster (or set of clusters, depending on the topology used).</dd>

  <dt>Nodes</dt>
  <dd>You can think of these as "container clients". These are the individual hosts (physical or virtual) that Docker is installed on and hosts the various containers within your managed cluster.</dd>
  <dd>Each node will run etcd (a key-pair management and communication service, used by Kubernetes for exchanging messages and reporting on cluster status) as well as the Kubernetes Proxy.</dd>
</dl>
