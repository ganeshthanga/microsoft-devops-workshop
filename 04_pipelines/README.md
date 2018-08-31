# Pipelines

A major benefit of adopting Kubernetes is the enhancements it brings to the Agile development lifecycle. We have all of these scriptable ways to deploy applications, it wouldn't make sense to skip out on automation.

The first section discusses a special tool called Helm which is essentially a Kubernetes package manager, that you can build packages for. It gives you history and rollback capabilities out of the box, for any deployment done via this tool.

We will also cover Jenkins, Immutable Builds, and Integration testing, to close out the workshop.

## Table of Contents

1. [Helm](01_helm)
   1. [Install Helm](01_helm/01_install_helm.md)
   2. [Reading Charts](01_helm/02_reading_charts.md)
   3. [Creating Charts](01_helm/03_creating_charts.md)
   4. [State Metrics](https://github.com/helm/charts/tree/master/stable/kube-state-metrics)
   5. [Ingress Controller](https://github.com/helm/charts/tree/master/stable/nginx-ingress)
   6. [ELK Stack (Logging)](https://github.com/helm/charts/tree/master/stable/fluentd-elasticsearch)
   7. [Jenkins Stack](https://github.com/helm/charts/tree/master/stable/jenkins)
2. [Jenkins on K8s](02_jenkins)
   1. [Install Jenkins](02_jenkins/01_install_jenkins.md)
3. [Containers as a Build Artifact / Promoting Builds](03_immutable_builds)
4. [Integration Testing Microservices](04_integration_testing_microservices)