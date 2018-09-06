# Standard Kube System Pods

When you run the command, `kubectl get pods -n kube-system`, you are viewing the pods in the `kube-system` namespace. This namespace is typically reserved for cluster level services, such as autoscalar, external dns, kube proxy, heapster, kube dns, kubernetes dashboard, etc.

## Standard

The standard cluster services that are generally available on all clusters, are `kube-proxy`, `kube-dns`.

In unmanaged installations, its common to find these components running as containers, on the cluster as well: `etcd`, `kube-apiserver`, `dns-controller`, `kube-controller-manager`, `kube-scheduler`

These services correspond to the components we described earlier in this section.

## AKS

In AKS, there are special services deployed to your cluster such as `omasagent`, `tunnelfront`, and `kube-svc-redirect`.

1. OMSAgent - This reports to the azure portal to provide diagnostic information, and for portal to communicate to your nodes.
2. Tunnelfront - This is a service azure uses to facilitate communication between the managed control plane and the nodes in your cluster.
3. kube-svc-redirect - This is service handles some additiona traffic redirection in the AKS environment.