# Networking

Kubernetes supports several CNIs, which can be enabled, at the time of instantiating your cluster.

K8s Documentation: https://kubernetes.io/docs/concepts/cluster-administration/networking/

Up to this point, we have referred to containers, but Kubernetes encapsulates one or more containers into a primitive called a Pod, which we'll discuss more in the following sections.

Kubernetes approaches networking somewhat differently than Docker does by default. There are 4 distinct networking problems to solve:

1. Highly-coupled container-to-container communications: Inter-Pod Traffic / Localhost.
2. Pod-to-Pod communications
3. Pod-to-Service communications: pods using a load balancer or "Service" to reach other pods.
4. External-to-Service communications: clusters using NodePorts via "Service" to route traffic to pods.

Each CNI handles this process in a slightly different way. The most popular open-source solutions are Flannel, Calico, and subsequently Canal.

## Flannel

A very simple overlay network that satisfies the Kubernetes requirements. This is the no-frills overlay typically used in small scale installations, as it has a small footprint, but many users at scale have reported success. If you don't need fine-grained policy control for inter-pod traffic, this may be the option for you! 

Flannel runs a small, single binary agent called flanneld on each host, and is responsible for allocating a subnet lease to each host out of a larger, preconfigured address space. Flannel uses either the Kubernetes API or etcd directly to store the network configuration, the allocated subnets, and any auxiliary data (such as the host's public IP). Packets are forwarded using one of several backend mechanisms including VXLAN and various cloud integrations.

Read More: https://github.com/coreos/flannel

**NOTE:** There is a NetworkPolicy kubernetes primitive, which seeks to enable pod firewalling the same way other resources are defined, and flannel does not support this. If you need this feature, please choose an appropriate CNI.

## Calico

Calico can be deployed without encapsulation or overlays to provide high-performance, high-scale data center networking. Calico also provides fine-grained, intent based network security policy for Kubernetes pods via its distributed firewall. It creates and manages a flat layer 3 network, assigning each workload a fully routable IP address.

Read More: https://docs.projectcalico.org/v3.2/introduction/

## Canal (Flannel + Calico)

Calico can also be run in policy enforcement mode in conjunction with other networking solutions such as Flannel, which is where Canal comes into play. It's relatively new on the scene, and the latest information can be found here (ensure the link doesn't have a deprecation warning): https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/flannel