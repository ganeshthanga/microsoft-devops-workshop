# Kubernetes Controller Manager

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/

**Note:** This is part of the Kubernetes "Control Plane" that is abstracted from the end-user for most managed platforms, such as AKS. This quick demonstration serves to help you understand the components, and the primary DevOps tasks associated.

The controller manager is a daemon that runs the main control loops within Kubernetes. Kubernetes uses control loops to regulate desired state vs current state. In a sense, these loops are always driving to the desired state. There are several controllers that are native to Kubernetes, such as: Replication, Namespace, Endpoints, and ServiceAccounts Controllers. It's common for Kubernetes users to define their own controllers, that can utilize custom metadata to perform action within and outside of the cluster.

# Kubernetes Scheduler

**Note:** This is part of the Kubernetes "Control Plane" that is abstracted from the end-user for most managed platforms, such as AKS. This quick demonstration serves to help you understand the components, and the primary DevOps tasks associated.

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/

The scheduler can be considered the nervous system of Kubernetes. It is responsible for scheduling workload according to policies defined with variable topology, affinity, performance, capacity, and more. These policies are defined and stored with the same logic that runs the containers, and can be managed via SCM very easily.

Since the Controler Manager is such a pivotal component, metrics can help point to issues in health and performance.  Metrics are provided via Golang runtime metrics: e.g. go_routine count and contoller specific metrics such as those from AWS,n GCE or Open Stack.  Meterics are emmited in Prometheus format and are human readable.  

# Kubelet

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/

The Kubelet is the Kubernetes cluster agent. It heart beats an http request to the API to confirm its membership, and the controllers tell it what work to schedule on it's host.

This service would share the hosts docker daemon, such that it can schedule and run containers, even if it's running in a container, on that host itself. 

The most common operations that a user might encounter with the kubelet, is the need to restart it, under certain conditions. Generally, issues wherein a single node becomes "Not Ready" for extended periods of time, or if a node disappears from the node list altogether, warrant a kubelet restart.


# Kube Proxy

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/

The Kubernetes network proxy runs on each node. This reflects workloads on each node and can do simple stream forwarding or round robin forwarding across a set of backends. There is an optional addon that provides cluster DNS for these cluster IPs. The user must create a service with the apiserver API to configure the proxy.
