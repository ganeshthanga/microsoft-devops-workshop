# Ingress Controllers / Rules

Note: The examples in the following sections have been borrowed heavily from the [official Kubernetes docs](https://kubernetes.io/docs/concepts/services-networking/ingress/).

## Overview of Ingress

An Ingress is an API object that manages external access to the services in a cluster, typically HTTP.

* An Ingress is a collection of rules that allow inbound connections to reach the cluster services
* Configure to expose services through:
  * External URLs
  * Load Balancers
  * SSL-terminated endpoints
  * Name-based virtual hosting

An Ingress sits at a level above a Service.

Remember the Service Types:
* ClusterIP
  * Service is reachable only from inside of the cluster.
* NodePort
  * Service is reachable through `<NodeIP>:NodePort` address.
* LoadBalancer
  * Service is reachable through an *external* load balancer mapped to `<NodeIP>:NodePort` address.

Note: Creating a Service of type LoadBalancer, where the external load balancer is created in a Public Cloud (e.g., Azure), is not a very efficient way to get your applications to "talk" to the outside world. This is because every Service will create a new external load balancer, which is obviously an expensive proposition.

## What is an Ingress

Typically, services and pods have IPs only routable by the cluster network. All traffic that ends up at an edge router is either dropped or forwarded elsewhere. Conceptually, this might look like:
```
    internet
        |
  ------------
  [ Services ]
```

An Ingress is a collection of rules that allow inbound connections to reach the cluster services.

```
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```
It can be configured to give services externally-reachable URLs, load balance traffic, terminate SSL, offer name based virtual hosting, and more. Users request ingress by POSTing the Ingress resource to the API server. An Ingress controller is responsible for fulfilling the Ingress, usually with a loadbalancer, though it may also configure your edge router or additional frontends to help handle the traffic in an HA manner.

## What is an Ingress Controller

Configuring a webserver or loadbalancer is harder than it should be. Most webserver configuration files are very similar. There are some applications that have weird little quirks that tend to throw a wrench in things, but for the most part you can apply the same logic to them and achieve a desired result.

The Ingress resource embodies this idea, and an Ingress controller is meant to handle all the quirks associated with a specific "class" of Ingress.

An Ingress Controller is a daemon, deployed as a Kubernetes Pod, that watches the apiserver's /ingresses endpoint for updates to the Ingress resource. Its job is to satisfy requests for Ingresses.

The default, maintained by Kubernetes, Ingress Controllers are: [GCE](https://github.com/kubernetes/ingress-gce/blob/master/README.md) and [nginx](https://github.com/kubernetes/ingress-nginx/blob/master/README.md) controllers.

**IMPORTANT** In order for the Ingress resource to work, the cluster must have an Ingress controller running. This is unlike other types of controllers, which typically run as part of the kube-controller-manager binary, and which are typically started automatically as part of cluster creation. Choose the ingress controller implementation that best fits your cluster, or implement a new ingress controller.

## Types of Ingress

### Single Service Ingress

There are existing Kubernetes concepts that allow you to expose a single Service, however, you can do so through an Ingress as well, by specifying a *default backend* with no rules.

```
service/networking/ingress.yaml  
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80
```

If you create the above Ingress using `kubectl create -f`, you should see something like:
```
$ kubectl get ingress test-ingress
NAME           HOSTS   ADDRESS        PORTS     AGE
test-ingress   *       52.168.9.176   80        59s
```
where `52.168.9.176` is the IP allocated by the Ingress controller to satisfy this Ingress. If you did not configure an Ingress controller, you would not get an IP associated with this Ingress.

### Simple fanout

As described previously, Pods within kubernetes have IPs only visible on the cluster network, so we need something at the edge accepting ingress traffic and proxying it to the right endpoints. This component is usually a highly available loadbalancer. An Ingress allows you to keep the number of loadbalancers down to a minimum. For example, a setup like:

```
foo.bar.com -> 52.168.9.176 -> / foo    s1:80
                               / bar    s2:80
```

would require an Ingress such as:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80
```

When you create the Ingress with `kubectl create -f`:

```
$ kubectl describe ingress test

Name:             test
Namespace:        default
Address:          52.168.9.176
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   s1:80 (10.8.0.90:80)
               /bar   s2:80 (10.8.0.91:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     22s                loadbalancer-controller  default/test
```

The Ingress controller will provision an implementation specific loadbalancer that satisfies the Ingress, as long as the services (`s1`, `s2`) exist. When it has done so, you will see the address of the loadbalancer at the Address field.

Note: You will need to create a default-http-backend Service, if one does not already exist in your cluster.

### Name-based virtual hosting

Name-based virtual hosts use multiple host names for the same IP address.

```
foo.bar.com --|              |-> foo.bar.com s1:80
              | 52.168.9.176 |
bar.foo.com --|              |-> bar.foo.com s2:80
```

The following Ingress tells the backing loadbalancer to route requests based on the Host header.
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
```

**Default Backends**: An Ingress with no rules, like the one shown in the previous section, sends all traffic to a single default backend. You can use the same technique to tell a loadbalancer where to find your website’s 404 page, by specifying a set of rules and a default backend. Traffic is routed to your default backend if none of the Hosts in your Ingress match the Host in the request header, and/or none of the paths match the URL of the request.

### TLS

You can secure an Ingress by specifying a secret that contains a TLS private key and certificate. Currently the Ingress only supports a single TLS port, 443, and assumes TLS termination. If the TLS configuration section in an Ingress specifies different hosts, they will be multiplexed on the same port according to the hostname specified through the SNI TLS extension (provided the Ingress controller supports SNI). The TLS secret must contain keys named `tls.crt` and `tls.key` that contain the certificate and private key to use for TLS, e.g.:
```
apiVersion: v1
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
kind: Secret
metadata:
  name: testsecret
  namespace: default
type: Opaque
```

Referencing this secret in an Ingress will tell the Ingress controller to secure the channel from the client to the load balancer using TLS:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: no-rules-map
spec:
  tls:
  - secretName: testsecret
  backend:
    serviceName: s1
    servicePort: 80
```

Note that there is a gap between TLS features supported by various Ingress controllers. You must refer to the documentation for your Ingress controller in order to understand how TLS works in your environment.

### Updating an Ingress

If you would like to add a new Host to an existing Ingress (like the one we created above), you can update the Ingress by editing the resource:
```
$ kubectl describe ingress test

Name:             test
Namespace:        default
Address:          52.168.9.176
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   s1:80 (10.8.0.90:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     35s                loadbalancer-controller  default/test

$ kubectl edit ingress test
```

That last command will open up a text editor in your shell (using your default editor; e.g., Vim, Emacs, Nano, etc.) with the existing YAML (of the current Ingress). Modify this YAML to include your new Host:
```
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
        path: /foo
  - host: bar.baz.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
        path: /foo
...
```

Saving the YAML will update the resource in the API server, which should tell the Ingress controller to reconfigure the loadbalancer.

```
$ kubectl describe ingress test

Name:             test
Namespace:        default
Address:          52.168.9.176
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   s1:80 (10.8.0.90:80)
  bar.baz.com
               /foo   s2:80 (10.8.0.91:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     45s                loadbalancer-controller  default/test
```

Note: You can achieve the same by invoking `kubectl replace -f` on a modified Ingress YAML file. In fact, this is a much better way, as you can keep track of changes in version/revision control.
