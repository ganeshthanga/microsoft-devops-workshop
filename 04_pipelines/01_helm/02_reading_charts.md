# Reading helm Charts

If you are going to install 3rd party developed charts, it helps to understand how they are to be read, which in turn will help you develop better helm charts.

When you run the command:

`helm search`

It lists off many charts that are available for immediate install. You can find the source code for all of these charts here:
https://github.com/helm/charts/tree/master/stable

Generally, each helm chart consists of the same directory structure.

- `templates` - This directory will contain the kubernetes resource templates, and some helper files.
  - `tests` - This directory may contain resources that are used to test any code changes to the chart, and to test deployed services, should you have the need. They can be triggered by running `helm test <release name>`
  - `NOTES.txt` - Mustache file that provides a friendly message after the chart has been installed. It can include further instructions, or details about what was setup, based on the values provided.
  - `_helpers.tpl` - Mustache file that allows you to define composite values, that can be referenced in your k8s resource templates.
  - `*.yaml` - These are the templatized kubernetes resources, that are interpolated with the values.yaml
- `.helmignore` - This is comparable to `.gitignore` and `.dockerignore` except for the helm context. When you run `helm package <chart directory>` files listed here will be excluded. 
- `Chart.yaml` - Metadata about the chart including the version, name, search words, maintainers, etc.
- `OWNERS` - This file is used to determine who can review changes, and who can approve changes, typically used for publicly supported charts.
- `README.md` - The most important piece of all, this markdown file will explain the values file, and how you should use each property, to achieve your goals.
- `values.yaml` - This is the default values file for the chart. Typically, you will copy this values file into your SCM, and make modifications, you deem necessary. Occasionally there are values files with different names, which are not the default, but are provided for your reference, and may contain additional information.

We deployed Redis earlier as a StatefulSet, but there may have been some best practices that were not followed, and/or we may want to have a supported deployment of redis, rather than trying to roll our own.

You can find a chart for redis here, lets take a look: https://github.com/helm/charts/tree/master/stable/redis

From a glance, it appears that this is a clustered master-slave setup, that relies on these pods to maintain a quorum in order to preserve data. This chart would be a good choice, if you are not going to be sunk, if the cluster goes down or if several nodes go down simultaneously. Since redis is typically used for cache, and/or less critical data, this is generally ok.

There is an alternative chart: https://github.com/helm/charts/tree/master/stable/redis-ha which uses a different deployment strategy, with additional components, to better provide ha.

Neither of these is suitable, if you intended on using redis with PersistentVolumes, as neither chart implements PVs for redis. You could fork these charts, add the command flags to ensure data is written to disk, and then add our volumeClaimTemplate to the stateful deployments in the chart. 

Of course doing these things makes the chart deviate from master and become unsupported. If the service you are working with is familiar to you, its just important, to understand the implications these resource pose, and what tools you might use to solve them.

## Deploying a Stable Chart from the Repo

Helm has an upsert command that will install or update the release, if its present.

`helm update --install <release name> <chart directory / repo chart> -f <values overrides>`

### Installing with default Values
`helm update --install ha-redis stable/redis`

### Installing with Custom Values

You can copy the values.yaml from the chart repo to a local file called, `local-redis-values.yml`, and make customizations. Your values will be merged with the default values, prior to the interpolation step.

`helm update --install ha-redis stable/redis -f local-redis-values.yml`