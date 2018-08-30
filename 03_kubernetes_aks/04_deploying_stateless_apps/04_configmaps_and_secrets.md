# ConfigMaps and Secrets

While deploying an application, we may need to pass such runtime parameters like configuration details, passwords, etc. For example, let's assume we need to deploy ten different applications for our customers, and, for each customer, we just need to change the name of the company in the UI. Instead of creating ten different Docker images for each customer, we can just use the template image and pass the customers' names as a runtime parameter. In such cases, we can use the ConfigMap API resource. Similarly, when we want to pass sensitive information, we can use the Secret API resource. Think ''Secrets'' (for confidential data) and ''ConfigMaps'' (for non-confidential data).

[ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) allow you to decouple configuration artifacts from image content to keep containerized applications portable. Using ConfigMaps, we can pass configuration details as key-value pairs, which can be later consumed by Pods or any other system components, such as controllers. We can create ConfigMaps in two ways:

* From literal values; and
* From files.

## ConfigMaps

* Create a ConfigMap:
```
$ kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
configmap "my-config" created
```

* Get details on the above created ConfigMap:
```
$ kubectl get configmaps my-config -o yaml

apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: 2018-01-11T23:57:44Z
  name: my-config
  namespace: default
  resourceVersion: "117110"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: 37a43e39-f72b-11e7-8370-08002721601f

$ kubectl describe configmap/my-config

Name:         my-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
key2:
----
value2
key1:
----
value1
Events:  <none>
```

### Create a ConfigMap from a configuration file

* Create ConfigMap from the CLI:
```
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: customer1
data:
  TEXT1: Customer1_Company
  TEXT2: Welcomes You
  COMPANY: Customer1 Company Technology, LLC.
EOF
```

We can get the values of the given key as environment variables inside a Pod. In the following example, while creating the Deployment, we are assigning values for environment variables from the customer1 ConfigMap:
```
....
 containers:
      - name: my-app
        image: foobar
        env:
        - name: MONGODB_HOST
          value: mongodb
        - name: TEXT1
          valueFrom:
            configMapKeyRef:
              name: customer1
              key: TEXT1
        - name: TEXT2
          valueFrom:
            configMapKeyRef:
              name: customer1
              key: TEXT2
        - name: COMPANY
          valueFrom:
            configMapKeyRef:
              name: customer1
              key: COMPANY
....
```

With the above, we will get the <code>TEXT1</code> environment variable set to <code>Customer1_Company</code>, <code>TEXT2</code> environment variable set to <code>Welcomes You</code>, and so on.

We can also mount a ConfigMap as a Volume inside a Pod. For each key, we will see a file in the mount path and the content of that file become the respective key's value. For details, see [here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#adding-configmap-data-to-a-volume).

You can also use ConfigMaps to configure your cluster to use, as an example, 8.8.8.8 and 8.8.4.4 as its upstream DNS server:
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-dns
  namespace: kube-system
data:
  upstreamNameservers: |
    ["8.8.8.8", "8.8.4.4"]
```

## Secrets

Objects of type [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys. Putting this information in a Secret is safer and more flexible than putting it verbatim in a pod definition or in a docker image.

As an example, assume that we have a Wordpress blog application, in which our <code>wordpress</code> frontend connects to the MySQL database backend using a password. While creating the Deployment for <code>wordpress</code>, we can put the MySQL password in the Deployment's YAML file, but the password would not be protected. The password would be available to anyone who has access to the configuration file.

In situations such as the one we just mentioned, the Secret object can help. With Secrets, we can share sensitive information like passwords, tokens, or keys in the form of key-value pairs, similar to ConfigMaps; thus, we can control how the information in a Secret is used, reducing the risk for accidental exposures. In Deployments or other system components, the Secret object is ''referenced'', without exposing its content.

It is important to keep in mind that the Secret data is stored as plain text inside etcd. Administrators must limit the access to the API Server and etcd.

To create a Secret using the `kubectl create secret` command, we need to first create a file with a password, and then pass it as an argument.

* Create a file with your MySQL password:
```
$ echo mysqlpasswd | tr -d '\n' > password.txt
```

* Create the Secret:
```
$ kubectl create secret generic mysql-passwd --from-file=password.txt
```

* Describe the Secret:
```
$ kubectl describe secret/mysql-passwd

Name:         mysql-passwd
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  11 bytes
```

We can also create a Secret manually, using the YAML configuration file. With Secrets, each object data must be encoded using base64. If we want to have a configuration file for our Secret, we must first get the base64 encoding for our password:
```
$ cat password.txt | base64
bXlzcWxwYXNzd2Q==
```

and then use it in the configuration file:
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-passwd
type: Opaque
data:
  password: bXlzcWxwYXNzd2Q=
```

Note that base64 encoding does not do any encryption and anyone can easily decode it:
```
$ echo "bXlzcWxwYXNzd2Q=" | base64 -d  # => mysqlpasswd
```

Therefore, make sure you do not commit a Secret's configuration file in the source code.

We can get Secrets to be used by containers in a Pod by mounting them as data volumes, or by exposing them as environment variables.

We can reference a Secret and assign the value of its key as an environment variable (<code>WORDPRESS_DB_PASSWORD</code>):
```
.....
    spec:
      containers:
      - image: wordpress:4.7.3-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-password
              key: password.txt
.....
```

Or, we can also mount a Secret as a Volume inside a Pod. A file would be created for each key mentioned in the Secret, whose content would be the respective value. See [here](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod) for details.

