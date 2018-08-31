# Namespaces

If we have numerous users whom we would like to organize into teams/projects, we can partition the Kubernetes cluster into sub-clusters using *[Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)*. The names of the resources/objects created inside a Namespace are unique, but not across Namespaces.

To list all the Namespaces, we can run the following command:
```
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    7d
kube-public   Active    7d
kube-system   Active    7d
```

Generally, Kubernetes creates two default namespaces: `kube-system` and `default`. The `kube-system` namespace contains the objects created by the Kubernetes system. The `default` namespace contains the objects which belong to any other namespace. By default, we connect to the `default` Namespace.

`kube-public` is a special namespace, which is readable by all users and used for special purposes, like bootstrapping a cluster. 

Using *[Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)*, we can divide the cluster resources within Namespaces.

## Creating custom Namespaces

You can create your own custom Namespaces. You can use Namespaces to divide your Kubernetes components into teams, departments, or for your Development, QA, Staging, Production, etc. environments.

For this demo, we will create two additional Kubernetes namespaces to hold our content.

Imagine a scenario where an organization is using a shared Kubernetes cluster for development and production use cases.

The development team would like to maintain a space in the cluster where they can get a view on the list of Pods, Services, and Deployments they use to build and run their application. In this space, Kubernetes resources come and go, and the restrictions on who can or cannot modify resources are relaxed to enable agile development.

The operations team would like to maintain a space in the cluster where they can enforce strict procedures on who can or cannot manipulate the set of Pods, Services, and Deployments that run the production site.

One pattern this organization could follow is to partition the Kubernetes cluster into two namespaces: development and production.

Let's create two new namespaces to hold our work.

* Create a Namespace called "dev":
```
$ kubectl create namespace dev
namespace/dev created

$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    7d
dev           Active    3s
kube-public   Active    7d
kube-system   Active    7d
```

However, this is not a good way of creating Namespaces, as all of the commands you typed in the CLI to create the Namespace a lost in your history and their is no revision/version control.

* Delete the above Namespace
$ kubectl delete namespace dev

A much better way of creating a Namespace (like any other object in Kubernetes) is to create a YAML file and store that in a repository (e.g., GitHub):
```
$ cat << EOF >namespace-dev-create.yml
kind: Namespace
apiVersion: v1
metadata:
  name: dev
  labels:
    name: development
EOF

$ cat << EOF >namespace-prod-create.yml
kind: Namespace
apiVersion: v1
metadata:
  name: prod
  labels:
    name: production
EOF

$ kubectl create --validate -f namespace-dev-create.yml 
namespace/dev created

$ kubectl create --validate -f namespace-prod-create.yml
namespace/prod created

$ kubectl get ns --show-labels
NAME          STATUS    AGE       LABELS
default       Active    7d        <none>
dev           Active    13s       name=development
prod          Active    9s        name=production
```

## Create Pods in a given namespace

A Kubernetes namespace provides the scope for Pods, Services, and Deployments in the cluster.

Users interacting with one namespace do not see the content in another namespace.

To demonstrate this, let's spin up a simple Deployment and Pods in the development namespace.

We first check what is the current context:
```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/xtof/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/xtof/.minikube/client.crt
    client-key: /home/xtof/.minikube/client.key

$ kubectl config current-context 
minikube
```

The next step is to define a context for the kubectl client to work in each namespace. The value of "cluster" and "user" fields are copied from the current context.

```
$ kubectl config set-context dev --namespace=dev --cluster=minikube --user=minikube
Context "dev" created.

$ kubectl config set-context prod --namespace=prod --cluster=minikube --user=minikube
Context "prod" created.

By default, the above commands adds two contexts that are saved into file .kube/config. You can now view the contexts and alternate against the two new request contexts depending on which namespace you wish to work against.

To view the new contexts:
```
$ kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/xtof/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    namespace: dev
    user: minikube
  name: dev
- context:
    cluster: minikube
    user: minikube
  name: minikube
- context:
    cluster: minikube
    namespace: prod
    user: minikube
  name: prod
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/xtof/.minikube/client.crt
    client-key: /home/xtof/.minikube/client.key
```

Now, switch to operate in the development namespace:
```
$ kubectl config use-context dev
Switched to context "dev".
```

Verify that you are in this namespace:
```
$ kubectl config current-context
dev
```

Deploy a simple Nginx app in this new development namespace:
```
$ kubectl run nginx --image=nginx --port=80

$ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/nginx-768979984b-7v8x7   1/1       Running   0          3m

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1         1         1            1           3m

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-768979984b   1         1         1         3m
```

Now, switch to the production namespace:
```
$ kubectl config use-context prod

$ kubectl get all
No resources found.
```

As you can see, all resources in one namespace are hidden from all others.
