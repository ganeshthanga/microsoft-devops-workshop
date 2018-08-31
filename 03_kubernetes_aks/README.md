# Kubernetes

Kubernetes (also known by its numeronym k8s) is an open source container cluster manager. Kubernetes' primary goal is to provide a platform for automating deployment, scaling, and operations of application containers across a cluster of hosts. Kubernetes was released by Google on July 2015.

Latest stable release: v1.11.1 (2018-08-17)

Get the latest stable release with:
```
$ curl -sSL https://dl.k8s.io/release/stable.txt
```

Kubernetes is built through the definition of a set of components (building blocks or "primitives") which, when used collectively, provide a method for the deployment, maintenance, and scalability of container-based application clusters.

These "primitives" are designed to be loosely coupled (i.e., where little to no knowledge of the other component definitions is needed to use) as well as easily extensible through an API. Both the internal components of Kubernetes as well as the extensions and containers make use of this API.

## Managed Kubernetes (AKS)

In this workshop, we will discuss how to setup Kubernetes clusters via the Azure AKS service. In a managed cluster, many of the control-plane components are abstracted from the end-user. This means that Azure provides an SLA for the control-plane and leaves you to focus on deploying and managing your workloads.

Because you won't have access to the nodes running etcd and the api, there are some customizations that may not be possible, at that level. 

## Table of Contents

1. [Overview / Moving Parts](01_overview)
   1. [ETCD](01_overview/01_etcd.md)
   2. [Kubernetes API](01_overview/02_kubernetes_api.md)
   3. [Controller Manager / Scheduler / Kubelet / Proxy](01_overview/03_controller_manager-scheduler-kubelet-proxy.md)
   4. [Networking (Flannel/Calico)](01_overview/04_networking.md)
   5. [Primitives (Deployments / Services / Pods / etc)](01_overview/05_k8s_primitives.md)
   6. [RBAC / Service Accounts](01_overview/06_rbac.md)
   7. [Cloud Provider](01_overview/07_cloud_provider.md)
2. [Environment Strategies (NonProd/Prod vs Dev/Test/Cert/Prod)](02_environment_strategies)
   1. [Namespaces](02_environment_strategies/01_namespaces.md)
   2. [Environments](02_environment_strategies/02_environments.md)
   3. [Service Discovery / KubeDNS](02_environment_strategies/03_service_discovery.md)
   4. [Workloads](02_environment_strategies/04_workloads.md)
3. [Standing up your First Cluster](03_standing_up_your_first_cluster)
   1. [Minikube (Local)](03_standing_up_your_first_cluster/01_minikube.md)
   2. AKS
   3. [kubectl](03_standing_up_your_first_cluster/03_kubectl.md)
   4. [Labels, Selectors, and Annotations](03_standing_up_your_first_cluster/04_labels_selectors_annotations.md)
   5. [Private Repo Image Pull Secrets](03_standing_up_your_first_cluster/05_private_repo_image_pull_secrets.md)
4. [Deploying Stateless Applications](04_deploying_stateless_apps)
   1. [Deployments](04_deploying_stateless_apps/01_deployments.md)
   2. [Jobs](04_deploying_stateless_apps/02_jobs.md)
   3. [DaemonSets](04_deploying_stateless_apps/03_daemon_sets.md)
   4. [ConfigMaps and Secrets](04_deploying_stateless_apps/04_configmaps_and_secrets.md)
5. Deploying Stateful Applications
   1. [Volume Management](05_deploying_stateful_apps/01_volume_management.md)
   2. [Stateful Sets](05_deploying_stateful_apps/02_stateful_sets.md)
6. System Services
   1. Standard Kube-System Pods
   2. Third Party Add-Ons (Datadog, Helm)
   3. Developing from Scratch
7. [Ingress / Traffic Routing](07_ingress_traffic_management)
   1. [Defining Services (ClusterIP / NodePort / LoadBalancer)](07_ingress_traffic_management/01_defining_services.md)
   2. [Ingress Controllers / Rules](07_ingress_traffic_management/02_ingress_controllers_rules.md)
8. Auto-Scaling / SRE
   1. Resources (Limits / Requests)
   2. Horizontal Pod Autoscaling
   3. Cluster Scaling
   4. Liveness / Readiness Probes
9. [Revisiting Section 2.3.4 .NET Core - ReactJS](09_revisiting_netcore_react)
   1. [Setting Up Redis](09_revisiting_netcore_react/01_setting_up_redis.md)
   2. [Setting Up Our Web App](09_revisiting_netcore_react/02_setting_up_our_web_app.md)
