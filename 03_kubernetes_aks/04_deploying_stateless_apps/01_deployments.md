# Deployments

A *[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)* controller provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

## Creating a Deployment

![Deployment](img/deployment_simple.png)

The following is an example of a Deployment. It creates a ReplicaSet to bring up three [Nginx](https://hub.docker.com/_/nginx/) Pods:
```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

* Check the syntax of the Deployment (YAML):
```
$ kubectl create -f nginx-deployment.yml --dry-run
deployment.apps/nginx-deployment created (dry run)
```

* Create the Deployment:
```
$ kubectl create -f nginx-deployment.yml --record
deployment "nginx-deployment" created
```
Note: By appending `--record` to the above command, we are telling the API to record the current command in the annotations of the created or updated resource. This is useful for future review, such as investigating which commands were executed in each Deployment revision.

* Get information about our Deployment:
```
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           24s
```

```
$ kubectl describe deployment/nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sat, 25 Aug 2018 16:39:27 -0700
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=1
                        kubernetes.io/change-cause=kubectl create --filename=nginx-deployment.yml --record=true
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-75675f5897 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  1m    deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
```

* Get information about the ReplicaSet created by the above Deployment:
```
$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-75675f5897   3         3         3         2m
```

```
$ kubectl describe replicaset/nginx-deployment-75675f5897
Name:           nginx-deployment-75675f5897
Namespace:      default
Selector:       app=nginx,pod-template-hash=3123191453
Labels:         app=nginx
                pod-template-hash=3123191453
Annotations:    deployment.kubernetes.io/desired-replicas=3
                deployment.kubernetes.io/max-replicas=4
                deployment.kubernetes.io/revision=1
                kubernetes.io/change-cause=kubectl create --filename=nginx-deployment.yml --record=true
Controlled By:  Deployment/nginx-deployment
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
           pod-template-hash=3123191453
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  3m    replicaset-controller  Created pod: nginx-deployment-75675f5897-rnjhz
  Normal  SuccessfulCreate  3m    replicaset-controller  Created pod: nginx-deployment-75675f5897-ckwl9
  Normal  SuccessfulCreate  3m    replicaset-controller  Created pod: nginx-deployment-75675f5897-wvvgh
```

* Get information about the Pods created by this Deployment:
```
$ kubectl get pods --show-labels -l app=nginx -o wide
NAME                                READY     STATUS    RESTARTS   AGE       IP           NODE       LABELS
nginx-deployment-75675f5897-ckwl9   1/1       Running   0          3m        172.17.0.5   minikube   app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-rnjhz   1/1       Running   0          3m        172.17.0.6   minikube   app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-wvvgh   1/1       Running   0          3m        172.17.0.4   minikube   app=nginx,pod-template-hash=3123191453
```

## Updating a Deployment

![Deployment Update](img/deployment_update.png)

Note: A Deployment's rollout is triggered if, and only if, the Deployment's pod template (that is, `.spec.template`) is changed (for example, if the labels or container images of the template are updated). Other updates, such as scaling the Deployment, do not trigger a rollout.

Suppose that we want to update the Nginx Pods in the above Deployment to use the `nginx:1.9.1` image instead of the `nginx:1.7.9` image.

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated
```

Alternatively, we can edit the Deployment and change `.spec.template.spec.containers[0].image` from `nginx:1.7.9` to `nginx:1.9.1`:

```
$ kubectl edit deployment/nginx-deployment
deployment "nginx-deployment" edited
```

* Check on the rollout status:
```
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```

* Get information about the updated Deployment:
```
$ kubectl get deploy
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           7m
 
$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-75675f5897   0         0         0         8m   # <- old ReplicaSet using nginx:1.7.9
nginx-deployment-c4747d96c    3         3         3         57s  # <- new ReplicaSet using nginx:1.9.1

$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION  CHANGE-CAUSE
1         kubectl create --record=true --filename=nginx-deployment.yml
2         kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1

$ kubectl rollout history deployment/nginx-deployment --revision=2
deployments "nginx-deployment" with revision #2
Pod Template:
  Labels:	app=nginx
	pod-template-hash=703038527
  Annoations:	kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
  Containers:
    nginx:
     Image:	nginx:1.9.1
     Port:	80/TCP
     Environment:	<none>
     Mounts:	<none>
  Volumes:	<none>
```

## Rolling back to a previous revision

Undo the current rollout and rollback to the previous revision:
```
$ kubectl rollout undo deployment/nginx-deployment
deployment "nginx-deployment" rolled back
```

Alternatively, you can rollback to a specific revision by specify that in `--to-revision`:
```
$ kubectl rollout undo deployment/nginx-deployment --to-revision=1
deployment "nginx-deployment" rolled back
```
