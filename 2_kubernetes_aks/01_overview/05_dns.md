# Kubernetes DNS

There is a Kubernetes DNS pod and service that is pre-defined on the cluster, if enabled during instantiation. Every Service defined in the cluster (including the DNS server itself) is assigned a DNS name. By default, a client Pod’s DNS search list will include the Pod’s own namespace and the cluster’s default domain.

For example: `<service-name>.<namespace>.svc.cluster.local`

Any other pods that share that namespace, could query this service, simply by using the `<service-name>` as the hostname. Cross name-space communication requires full dns names, as specified above.

This makes isolating environments for developers, qa, product owners, etc. very easy to read, consume, understand, and implement. 