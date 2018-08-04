# microsoft-devops-workshop
Code for the demos and labs for this workshop

# Table of Contents

1. Introduction
   1. Requirements / Assumptions
   2. Goals of the Workshop
1. Docker
   1. Benefits of Containers
   2. Containerization
      1. Java [JVM Specifics]
      2. Ruby
      3. NodeJs
      4. Python
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
   2. Environment Strategies (NonProd/Prod vs Dev/Test/Cert/Prod)
      1. Namespacing
      2. Service Discovery / KubeDNS
   3. Deploying Stateless Applications
      1. Hello-World
   4. Deploying Stateful Applications (w/ PVC)
      1. ELK Stack
   5. Defining Services ( ClusterIP / NodePort / LoadBalancer )
   6. Ingress / Traffic Routing
   7. Auto-Scaling
      1. Resources (Limits / Requests)
      2. Horizontal Pod Autoscaling
      3. Cluster Scaling
3. Pipelines
   1. Jenkins
   2. Containers as a Build Artifact / Promoting Builds
   3. Helm
   4. Integration Testing Micro-Services
