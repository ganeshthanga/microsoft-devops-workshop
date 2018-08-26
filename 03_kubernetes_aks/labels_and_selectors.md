# Labels and Selectors

*[Labels https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)* are key-value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each object can have a set of key-value labels defined. Each key must be unique for a given object.

```
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

## Syntax and character set

Labels are key-value pairs. Valid label keys have two segments: an optional prefix and name, separated by a slash (`/`). The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character (`[a-z0-9A-Z]`) with dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between. The prefix is optional. If specified, the prefix must be a DNS subdomain: a series of DNS labels separated by dots (`.`), not longer than 253 characters in total, followed by a slash (`/`). If the prefix is omitted, the label key is presumed to be private to the user. Automated system components (e.g. kube-scheduler, kube-controller-manager, kube-apiserver, kubectl, or other third-party automation) which add labels to end-user objects must specify a prefix. The `kubernetes.io/` prefix is reserved for Kubernetes core components.

Valid label values must be 63 characters or less and must be empty or begin and end with an alphanumeric character (`[a-z0-9A-Z]`) with dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between.

## Label selectors

Unlike names and UIDs, labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

Via a label selector, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.

The API currently supports two types of selectors: equality-based and set-based. A label selector can be made of multiple requirements which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical AND (`&&`) operator.

An empty label selector (that is, one with zero requirements) selects every object in the collection.

A null label selector (which is only possible for optional selector fields) selects no objects.

Note: the label selectors of two controllers must not overlap within a namespace, otherwise they will fight with each other.
Note that labels are not restricted to pods. You can apply them to all sorts of objects, such as nodes or services.

## Examples

* Label a given node:
```
$ kubectl label node k8s.worker1.local network=gigabit
```

* With "Equality-based", one may write:
```
$ kubectl get pods -l environment=production,tier=frontend
```

* Using "set-based" requirements:
```
$ kubectl get pods -l 'environment in (production),tier in (frontend)'
```

* Implement the OR operator on values:
```
$ kubectl get pods -l 'environment in (production, qa)'
```

* Restricting negative matching via exists operator:
```
$ kubectl get pods -l 'environment,environment notin (frontend)'
```

* Show the current labels on your pods:
```
$ kubectl get pods --show-labels
NAME      READY     STATUS    RESTARTS   AGE       LABELS
busybox   1/1       Running   25         9d        <none>
nfs-pod   1/1       Running   16         6d        name=nfs-pod
```

* Add a label to an already running/existing pod:
```
$ kubectl label pods busybox owner=christoph
pod "busybox" labeled

$ kubectl get pods --show-labels
NAME      READY     STATUS    RESTARTS   AGE       LABELS
busybox   1/1       Running   25         9d        owner=christoph
nfs-pod   1/1       Running   16         6d        name=nfs-pod
```

* Select a pod by its label:
```
$ kubectl get pods --selector owner=christoph
#~OR~
$ kubectl get pods -l owner=christoph
NAME      READY     STATUS    RESTARTS   AGE
busybox   1/1       Running   25         9d
```

* Delete/remove a given label from a given pod:
```
$ kubectl label pod busybox owner-
pod "busybox" labeled

$ kubectl get pods --show-labels
NAME      READY     STATUS    RESTARTS   AGE       LABELS
busybox   1/1       Running   25         9d        <none>
```

* Get all pods that belong to both the `production` *and* the `development` environments:
```
$ kubectl get pods -l 'env in (production, development)'
```

### Using Labels to select a Node on which to schedule a Pod

* Label a Node that uses SSDs as its primary HDD:
```
$ kubectl label node k8s.worker1.local hdd=ssd
```

# Annotations

With *[Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)*, we can attach arbitrary, non-identifying metadata to objects, in a key-value format:

```
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```

The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.

In contrast to Labels, annotations are not used to identify and select objects. Annotations can be used to:

* Store build/release IDs, which git branch, etc.
* Phone numbers of persons responsible or directory entries specifying where such information can be found
* Pointers to logging, monitoring, analytics, audit repositories, debugging tools, etc.
* Etc.

For example, while creating a Deployment, we can add a description like the one below:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver
  annotations:
    description: Deployment based PoC dates 24 August 2018
....
....
```

We can look at annotations while describing an object:

```
$ kubectl describe deployment webserver
Name:                webserver
Namespace:           default
CreationTimestamp:   Fri, 24 Aug 2018 13:18:23 -0800
Labels:              app=webserver
Annotations:         deployment.kubernetes.io/revision=1
                     description=Deployment based PoC dates 24 August 2018
...
...
```
