# Daemon Sets
In some cases, like collecting monitoring data from all nodes, or running a storage daemon on all nodes, etc., we need a specific type of Pod running on all nodes at all times. A *[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)* is the object that allows us to do just that. 

Whenever a node is added to the cluster, a Pod from a given DaemonSet is created on it. When the node dies, the respective Pods are garbage collected. If a DaemonSet is deleted, all Pods it created are deleted as well.

Example DaemonSet:
```
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: pause-ds
spec:
  selector:
    matchLabels:
      quiet: "pod"
  template:
    metadata:
      labels:
        quiet: pod
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: pause-container
        image: k8s.gcr.io/pause:2.0
```
