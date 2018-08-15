# Microsoft DevOps Workshop
Code for the demos and labs for this workshop

by Jerry Meisner and Christoph Champ for Redapt, Inc. (August 2018)

# Table of Contents

1. Introduction
   1. Requirements / Assumptions
   2. Goals of the Workshop
1. Docker
   1. Benefits of Containers
   2. Containerization
      1. Java [JVM Specifics]
      2. .NET (Windows Base Containers)
      3. Ruby
      4. NodeJs
      5. Python
   3. Configurability
   4. Security
2. Kubernetes / AKS
   1. Overview / Moving Parts
      1. ETCD
      2. Kubelet
      3. Kubernetes API
      4. Networking (Flannel/Calico)
      5. DNS (Internal / External)
      6. Primitives (Deployments / Services / Pods / etc)
      7. RBAC / Service Accounts
      8. Standing up your First Cluster in AKS
   2. Environment Strategies (NonProd/Prod vs Dev/Test/Cert/Prod)
      1. Namespacing
      2. Service Discovery / KubeDNS
   3. Deploying Stateless Applications
      1. Hello-World
      2. Config Maps / Secrets
   4. Deploying Stateful Applications
      1. ELK Stack
      2. PVC
      3. Volume Types
   5. System Services
      1. Standard Kube-System Pods
      2. Third Party Add-Ons (Datadog, Helm)
      3. Developing from Scratch
   6. Ingress / Traffic Routing
      1. Defining Services ( ClusterIP / NodePort / LoadBalancer )
      2. Ingress Controllers / Rules
   7. Auto-Scaling / SRE
      1. Resources (Limits / Requests)
      2. Horizontal Pod Autoscaling
      3. Cluster Scaling
      4. Liveness / Readiness Probes
3. Pipelines
   1. Jenkins
   2. Containers as a Build Artifact / Promoting Builds
   3. Helm
   4. Integration Testing Micro-Services
