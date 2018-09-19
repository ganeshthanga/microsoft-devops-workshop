# Private Repo Image Pull Secrets

In Kubernetes, we use a special command to create an `ImagePullSecret` that we can reference in our deployments that use private images.

```
$ kubectl create secret docker-registry regcred \
    --docker-server=<your-registry-server> \
    --docker-username=<your-name> \
    --docker-password=<your-pword> \
    --docker-email=<your-email>
```

The above will automatically create a secret named `docker-registry`, which you could add to `imagePullSecrets` as part of your k8s resource definitions. The contents of this secret is a paramaterized `~/.docker/config.json` file.

Example deployment with `imagePullSecret` defined:

```
apiVersion: apps/v1 
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
        image: myprivaterepo/nginx:1.7.9
        ports:
        - containerPort: 80
    imagePullSecrets:
    - name: docker-registry
```

This could be applied via `kubectl`, and will work if the command at the top of this readme is run first.

This secret can also be applied the same way to Pod definitions, StatefulSet definitions, DaemonSet definitions, etc. 
