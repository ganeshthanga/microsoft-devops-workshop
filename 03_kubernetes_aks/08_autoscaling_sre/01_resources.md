# Resources (Requests / Limits)

When deploying workloads to standard VMs you typically consider the host itself, and whether it is capable of handling all of the services that are running on it. 

In Kubernetes, we can define resource requests and limits in terms of CPU and Mem. As you have seen in our previous examples, these values are not required to simply perform deployments, by default Kubernetes will make attempts to divide the load among your nodes, using various rules and the number of pods assigned to each node. The pod will be unbounded, meaning it can consume as much as it wants, so long as that is available on the node. 

This is not ideal because you could deploy a resource intensive pod, which will consume the vast majority of resources, and the other pods could starve. 

You can avoid this scenario, by defining "requests", which is effectively the floor in the resource usage range. Defining requests puts the pod into a garunteed class, wherein there will always be at least that much dedicated to the pod, at all times. Kubernetes will place load on the nodes by looking at the requests, in order to make better decisions. If a Container exceeds its memory request, it is likely that its Pod will be evicted whenever the node runs out of memory.

Even with a minimum, there is a chance, your pod could consume all of the remaining un-"requested" resources, which could cause those containers, running unbounded to fail, due to the condition described above. That is where the "limit" comes into play. If a Container exceeds its memory or cpu limit, it might be terminated. If it is restartable, the kubelet will restart it, as with any other type of runtime failure.

These resource requests and limits can be specified individually, or altogether. The important thing, is that even if the Pod has more than one container, each container can have different requests and limits. The pod is ultimately scheduled based on the sum, and whats available.


```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```