# Stateful Sets

When you are persisting data, it makes sense to take a more methodical approach when scaling up and scaling down, as to ensure data is not lost, based on our architecture. A *[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)* is the object that allows us to do that. 

The SatefulSet controller monitors and ensures the required number of Pods are running, replacing Pods that die. The stateful sets controller follows different rules than the standard replica set controller. When it brings up new instances, each is appended with a number, indicating the order it was created. When the number of pods are reduced it destroys the newest instances first, generally scale down is more gated and throttled for these sets as well.

Example StatefulSet:
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
