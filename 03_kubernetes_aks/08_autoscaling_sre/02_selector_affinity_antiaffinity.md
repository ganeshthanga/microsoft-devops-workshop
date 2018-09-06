# Affinity / Anti-Affinity

In short, as part of your pod definition, you can designate that pods and/or nodes are attracted or repelled by eachother, during scheduling. These come in `preferred` or `required` varieties, that amount to setting a hard and soft requirement.

## NodeSelector

The NodeSelector attribute can be used to specify that your pods, should only be schedule to nodes that match a certain label, or set of labels.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

## Affinity

Pods can have affinity for other pods or affinity for certain types of nodes.

### Pod Affinity/Anti-Affinity

The following pod "prefers" to be scheduled in the same zone as pods that have `your-label-key` specified as `S1`.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: your-label-key
            operator: In
            values: [“S1”]
        topologyKey: failure-domain.beta.kubernetes.io/zone
```

The following pod prefers to be schedule on nodes that do not already have an instance of this pod on it.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    release: test
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchExpressions:
              - key: release
                operator: In
                values:
                  - test
```

### Node Affinity/Anti-Affinity

The following pod "prefers" to be scheduled onto nodes that do not have the `failure-domain.beta.kubernetes.io/zone` label specified as `us-central1-a`. You can replace `NotIn` for `In` to achieve the opposite.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: "failure-domain.beta.kubernetes.io/zone"
              operator: NotIn
              values: ["us-central1-a"]
```


#### Taints / Tolerations

You also have the ability to `taint` particular nodes, with certain labels. This will mark the node as unschedulable by any pods that do not have a toleration for taint with key `your-key`. This means that the node will remain empty until you define a deployment or pod manager that includes tolerations that allow it to be on those tainted nodes.

ie: `kubectl taint nodes node1 your-key=value:NoSchedule`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "your-key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```
