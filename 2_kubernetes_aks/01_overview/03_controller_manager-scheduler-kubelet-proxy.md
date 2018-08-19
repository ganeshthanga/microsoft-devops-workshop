# Kubernetes Controller Manager

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/

**Note:** This is part of the Kubernetes "Control Plane" that is abstracted from the end-user for most managed platforms, such as AKS. This quick demonstration serves to help you understand the components, and the primary DevOps tasks associated.



# Kubernetes Scheduler

**Note:** This is part of the Kubernetes "Control Plane" that is abstracted from the end-user for most managed platforms, such as AKS. This quick demonstration serves to help you understand the components, and the primary DevOps tasks associated.

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/



# Kubelet

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/

The Kubelet is the Kubernetes cluster agent. It heart beats an http request to the API to confirm its membership, and the controllers tell it what work to schedule on it's host.

This service would share the hosts docker daemon, such that it can schedule and run containers, even if it's running in a container, on that host itself. 

The most common operations that a user might encounter with the kubelet, is the need to restart it, under certain conditions. Generally, issues wherein a single node becomes "Not Ready" for extended periods of time, or if a node disappears from the node list altogether, warrant a kubelet restart.


# Kube Proxy

K8s Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/