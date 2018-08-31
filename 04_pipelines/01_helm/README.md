# Helm (https://helm.sh)

Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application.

Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste madness.

The latest version of Helm is maintained by the CNCF - in collaboration with Microsoft, Google, Bitnami and the Helm contributor community.

Each helm chart consists of templatized kubernetes primitives (deployments, services, volumes, etc), that are generated prior to deploy, based on a pre-defined set of values, that are shared between the resources/primitives. 

This makes managing multiple deployments and environments much simpler. Rather than having the full set of yml files for each individual deployment in scm, you can maintain one `instance-values.yml` file for each deployment. You can make changes to the chart, and then incrementally roll the changes out to your environments, in a controlled fashion.

- Helm has two parts: a client (helm) and a server (tiller)
- Tiller runs inside of your Kubernetes cluster, and manages releases (installations) of your charts.
- Helm runs on your laptop, CI/CD, or wherever you want it to run.
- Charts are Helm packages that contain at least two things:
  - A description of the package (Chart.yaml)
  - One or more templates, which contain Kubernetes manifest files
- Charts can be stored on disk, or fetched from remote chart repositories (like Debian or RedHat packages)

## Table of Contents

1. [Install Helm](01_install_helm.md)
2. [Reading Charts](02_reading_charts.md)
3. [Creating Charts](03_creating_charts.md)
4. [State Metrics](https://github.com/helm/charts/tree/master/stable/kube-state-metrics)
5. [Ingress Controller](https://github.com/helm/charts/tree/master/stable/nginx-ingress)
6. [ELK Stack (Logging)](https://github.com/helm/charts/tree/master/stable/fluentd-elasticsearch)
7. [Jenkins Stack](https://github.com/helm/charts/tree/master/stable/jenkins)