# Service Discovery

### Kubernetes DNS

There is a Kubernetes DNS pod and service that is pre-defined on the cluster, if enabled during instantiation. Every Service defined in the cluster (including the DNS server itself) is assigned a DNS name. By default, a client Pod’s DNS search list will include the Pod’s own namespace and the cluster’s default domain.

Every Service defined in the cluster (including the DNS server itself) is assigned a DNS name.

For example: `<service-name>.<namespace>.svc.cluster.local`
	or
For example: `<pod-ip>.<namespace>.pod.cluster.local`

Any other pods that share that namespace, could query this service, simply by using the `<service-name>` as the hostname. Cross name-space communication requires full dns names, as specified above.

This makes seperating environments for developers, qa, product owners, etc. very easy to read, consume, understand, and implement. 

#### Pod DNS Policies
“Default“: The Pod inherits the name resolution configuration from the node that the pods run on. The node `/etc/resolv.conf` should be updated searching on `*cluster.local` on kube-dns/CoreDNS or DNS will be exclusively internal.

“ClusterFirst“: Any DNS query that does not match the configured cluster domain suffix, such as “www.kubernetes.io”, is forwarded to the upstream nameserver inherited from the node, all other requests are first sent to kube-dns/CoreDNS for resolution. Cluster administrators may have extra stub-domain and upstream DNS servers configured.

“ClusterFirstWithHostNet“: For Pods running with hostNetwork.

“None“: It allows a Pod to ignore DNS settings from the Kubernetes environment. All DNS settings are supposed to be provided using the dnsConfig field in the Pod Spec. 
