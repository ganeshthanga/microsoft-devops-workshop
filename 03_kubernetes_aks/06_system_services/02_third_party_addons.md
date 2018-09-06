# Third Party Add Ons

In the old individual VM landscape, it made sense to get binaries from a third party service provider, and install them onto each machine they were needed on. 

In the Kubernetes landscape, these binaries are installed and run via containers. Those that need host file system access, get it via HostPath mounts, and those that need api access, can get it from a ServiceAccount, etc.
You should avoid customizing the nodes *unless you are creating a base image for a nodePool*, and opt for a strategy that takes advantage of volumeMounts and scheduling to deploy these services the same way you do your own containerized workloads.

For example, Helm (k8s package manager, we'll discuss later) has a local cli tool, but it also has a cluster service called "Tiller" that gets installed the first time you run `helm init`. 

Fluentd might be run on each node, to collect logs from all running containers, and forward them to a centralized data store.

The ingress controller interface has been implemented by nginx and haproxy, which both have a kubernetes deployment offering.

For Performance Monitoring and Alerting, A Prometheus stack could be launched. DataDog offers their installation strategy as a Kubernetes DaemonSet, where you just drop in your credentials, many of the third party SaaS addons function this way for Kubernetes support.

### Worth Mentioning
**Helm** is a package manager we will discuss later in the workshop. It's optional, but is widely adopted, and an excellent way to deploy common services and third party add-ons based on configuration values, rather than creating the kubernetes resources individually. 
