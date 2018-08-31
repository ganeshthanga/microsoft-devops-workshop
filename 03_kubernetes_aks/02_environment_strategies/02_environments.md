# Environments

Before you stand up your first cluster, it is important to understand how you are going to be using the cluster, and how you can best be prepared for the future scenarios.

Lets start by defining some of the conditions that most teams would like to achieve in terms of testability and reliability.

1. Zero Down Time Deployments to Production
2. All Changes are Tested separate without impact to Production
3. Each stage of the development pipeline should have its own environment / set of micro-services. (Dev/QA/Load/PO)
4. Owners should be able to manipulate their environments, but not anyone else's, production should be limited access.

#### Namespacing is Not Enough

A single cluster can be divided into seperate namespaces, and as previously discussed, can be controlled by access policies. It follows then that you could create namespaces such as `prod`, `qa`, `load`, and `po`, which could satisfy conditions 1, 3 & 4, but only partially satisfy condition 2. Since you are still sharing the same node pool(s) with all of the environments, theres a chance that changes introduced during testing could negatively impact the hosts, which are running production instances. 

Kubernetes provides mechanisms by which you could have deployments target specific nodes, to avoid scheduling test workloads onto the same hosts as production, but there is also a **constant stream of cluster-wide Kubernetes updates** that you'd want to test as part of QA and Dev before going to production, which disatisfies condition 2. Of course, if you are ok locking into a K8s version for the life of the cluster, which we do not recommend for any production scenario, that may not be a concern. 

#### Kubernetes Updates

It's recommended that most adopters start with at least 2 clusters, to separate `Prod` and `NonProd` workloads, primarily to gate updates to Kubernetes itself. You should try the latest in QA, which includes the formal update procedure to garuntee no downtime. If there is predicted down time because the updates are breaking changes or the procedure causes issues, you can plan accordingly and stand up new clusters and transfer `Prod` workloads onto them, and cut-over before bring down the old clusters.

#### Separate Clusters

If your organization is at the point of true large-scale `Load Testing`, that would facilitate a need to divide it from `NonProd` workloads. You would not want planned activities to impact developers and QA. 

Visualize this Namespace Strategy (Cluster - Namespaces):
`Prod` - `prod`
`NonProd` - `dev`, `qa`, `po`
`Load Testing` - `qa-load`

<Visualization>

#### Regional Availability

Beyond seperating workloads, you may want additional clusters per region, for High Availability and latency reduction. It's perfectly reasonable to have one cluster designated for `Load Testing` in a single region, and both a `Prod` and `NonProd` cluster in every supported region. This serves to allow developers to troubleshoot issues that may occur isolated in those regions. 

The `Prod` and `Load Testing` clusters should be comparable in how they are setup: instance size, number of namespaces, etc.

The `NonProd` clusters could be significantly smaller, simply testing integration with code and Kubernetes.

Organizations at this scale, might be interested in the [Multicluster Special Interest Group](https://github.com/kubernetes/community/tree/master/sig-multicluster), which has taken upon itself the, still beta, "Federated Control Plane" that would effectively allow you to simultaneously deploy workloads to all clusters in a given context. 

This is less applicable to managed Kubernetes (AKS, etc) users where the control planes are abstracted as a service. Generally, these service providers are roadmapping support, for cross-cluster operations, between clusters they manage for you.

<Visualization>
