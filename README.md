# Microsoft DevOps Workshop
Code for the demos and labs for this workshop

by Jerry Meisner and Christoph Champ for Redapt, Inc. (August 2018)

## Table of Contents

1. [Introduction](01_introduction)
1. [Docker](02_docker)
   1. [Overview](02_docker/01_overview)
      1. [Benefits of Containers](02_docker/01_overview/01_benefits_of_containers.md)
      2. [Install Docker](02_docker/01_overview/02_install_docker.md)
      3. [Docker commands](02_docker/01_overview/03_docker_commands.md)
      4. [Docker Directives](02_docker/01_overview/04_docker_directives.md)
      5. [Container Volume Management](02_docker/01_overview/05_container_volume_management.md)
      6. [Images](02_docker/01_overview/06_images.md)
      7. [Networking](02_docker/01_overview/07_networking.md)
      8. [Monitoring](02_docker/01_overview/08_monitoring.md)
      9. [Events](02_docker/01_overview/09_events.md)
      10. Docker Image Repositories (Public/Private)
   2. [Docker Compose](02_docker/02_docker_compose)
   3. [Containerization](02_docker/03_containerization)
       1. [Dependencies and Image Size](02_docker/03_containerization/01_dependencies_and_image_size.md)
       2. [Configurability and Security](02_docker/03_containerization/02_configurability_and_security.md)
       3. [Tagging and Versioning](02_docker/03_containerization/03_tagging_versioning.md)
       4. [.NET Core - ReactJS](02_docker/03_containerization/04_netcore_react.md)
       5. [Java Spring Boot](02_docker/03_containerization/05_java_spring.md)
       6. [Windows Containers](02_docker/03_containerization/06_windows.md)
2. [Kubernetes / AKS](03_kubernetes_aks)
   1. [Overview / Moving Parts](03_kubernetes_aks/01_overview)
      1. [ETCD](03_kubernetes_aks/01_overview/01_etcd.md)
      2. [Kubernetes API](03_kubernetes_aks/01_overview/02_kubernetes_api.md)
      3. [Controller Manager / Scheduler / Kubelet / Proxy](03_kubernetes_aks/01_overview/03_controller_manager-scheduler-kubelet-proxy.md)
      4. [Networking (Flannel/Calico)](03_kubernetes_aks/01_overview/04_networking.md)
      5. [Primitives (Deployments / Services / Pods / etc)](03_kubernetes_aks/01_overview/05_k8s_primitives.md)
      6. [RBAC / Service Accounts](03_kubernetes_aks/01_overview/06_rbac.md)
   2. [Environment Strategies (NonProd/Prod vs Dev/Test/Cert/Prod)](03_kubernetes_aks/02_environment_strategies)
      1. [Namespaces](03_kubernetes_aks/02_environment_strategies/01_namespaces.md)
      2. [Environments](03_kubernetes_aks/02_environment_strategies/02_environments.md)
      3. [Service Discovery / KubeDNS](03_kubernetes_aks/03_environment_strategies/02_service_discovery.md)
      4. [Workloads](03_kubernetes_aks/02_environment_strategies/04_workloads.md)
   3. [Standing up your First Cluster](03_kubernetes_aks/03_standing_up_your_first_cluster)
      1. [Minikube (Local)](03_kubernetes_aks/03_standing_up_your_first_cluster/01_minikube.md)
      2. AKS
      3. [kubectl](03_kubernetes_aks/03_standing_up_your_first_cluster/03_kubectl.md)
      4. [Labels, Selectors, and Annotations](03_kubernetes_aks/03_standing_up_your_first_cluster/04_labels_selectors_annotations.md)
   4. Deploying Stateless Applications
      1. [Deployments](03_kubernetes_aks/04_deploying_stateless_apps/01_deployments.md)
      2. [Jobs](03_kubernetes_aks/04_deploying_stateless_apps/02_jobs.md)
      3. [DaemonSets](03_kubernetes_aks/04_deploying_stateless_apps/03_daemon_sets.md)
      4. [ConfigMaps and Secrets](03_kubernetes_aks/04_deploying_stateless_apps/04_configmaps_and_secrets.md)
   5. Deploying Stateful Applications
      1. [Volume Management](03_kubernetes_aks/04_deploying_stateful_apps/01_volume_management.md)
      2. ELK Stack
   6. System Services
      1. Standard Kube-System Pods
      2. Third Party Add-Ons (Datadog, Helm)
      3. Developing from Scratch
   7. Ingress / Traffic Routing
      1. Defining Services (ClusterIP / NodePort / LoadBalancer)
      2. Ingress Controllers / Rules
   8. Auto-Scaling / SRE
      1. Resources (Limits / Requests)
      2. Horizontal Pod Autoscaling
      3. Cluster Scaling
      4. Liveness / Readiness Probes
3. Pipelines
   1. Jenkins
   2. Containers as a Build Artifact / Promoting Builds
   3. Helm
   4. Integration Testing Micro-Services
