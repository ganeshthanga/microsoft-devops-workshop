# Cloud Provider

In Kubernetes there is a concept of Cloud Provider, wherein there are many implementations of a common interface, that Kubernetes will use to make api calls to your designate cloud provider (AKS, AWS, GKE, etc), when certain actions are taken.

In general, when you setup your cluster for the first time, is when you setup your designated cloud provider, using flags on the go binaries. We wont cover too much on these providers as they are in a state of flux, so their integration point with Kubernetes is subject to change. More than likely, these will become services that you install, after bringing up your cluster, the first time, when the dust settles.

For more information:
https://github.com/kubernetes/kubernetes/tree/master/pkg/cloudprovider

What is important, and wont change is that the logic for creating and maintaining cloud disks, cloud load balancers, cloud instances, and in some cases, routes will be part of your designated cloud provider.

If you are interested in the implementation details, the various cloud providers are open source, and are useful in determining how actions are taken, on your behalf, based on each scenario.

1. [Azure](https://github.com/kubernetes/cloud-provider-azure)
2. [AWS](https://github.com/kubernetes/cloud-provider-aws)
3. [GKE](https://github.com/kubernetes/cloud-provider-gke)