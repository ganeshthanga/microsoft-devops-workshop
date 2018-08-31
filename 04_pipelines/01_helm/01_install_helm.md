# Install Helm

## Install the Helm Cli Tools
Helm comes with an additional command line tool, that functions similarly to kubectl, in that it is making calls against the kubernetes api.

You can retrieve the binary for your distribution, from their git release page:
https://github.com/helm/helm/releases

Or you can install it via package manager.

Homebrew users can use `brew install kubernetes-helm`.
Chocolatey users can use `choco install kubernetes-helm`.
GoFish users can use `gofish install helm`.

## Install Helm to Kubernetes

If you have RBAC enabled, and you should, you should create the following local file and then apply it.

`helm-tiller-sa.yml`
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

`kubectl apply -f helm-tiller-sa.yml`

`helm init --service-account tiller`

`helm search`

`helm repo update`