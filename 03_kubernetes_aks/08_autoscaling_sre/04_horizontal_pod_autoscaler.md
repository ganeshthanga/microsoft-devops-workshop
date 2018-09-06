# Horizontal Pod Auto Scaler

HPA is a Kubernetes primitive, that define cpu thresholds for automatically, horizontally scaling pods. In order for this to function correctly, the [default metrics server](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/metrics-server.md) must be deployed, to act as the data source. In AKS, this is already deployed for you, when you create your cluster.

![horizontal-pod-autoscaler](images/horizontal-pod-autoscaler.svg)

The following yaml, would scale the php-apache deployment, up to 10 pods, if the cpu utilization exceeds 50 percent.

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## Unschedulable Event

If you have appropriately defined resource requests, then adding additional pods automatically, might trigger an unschedulable event, notifying that the pod you are trying to add won't fit on the current cluster's nodes.

In the next section, we talk about scaling the cluster, in response to these events.