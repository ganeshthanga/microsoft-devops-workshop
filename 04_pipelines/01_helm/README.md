# Helm (https://helm.sh)

Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application.

Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste madness.

The latest version of Helm is maintained by the CNCF - in collaboration with Microsoft, Google, Bitnami and the Helm contributor community.

Each helm chart consists of templatized kubernetes primitives (deployments, services, volumes, etc), that are generated prior to deploy, based on a pre-defined set of values, that are shared between the resources/primitives. This makes managing multiple deployments and environments much simpler. Rather than having the full set of yml files for each individual deployment in scm, you can maintain one `instance-values.yml` file for each environment. You can make changes to the chart, and then incrementally roll the changes out to your environments, in a controlled fashion.