# kubectl

`kubectl` is a command line interface for running commands against Kubernetes clusters. This overview covers kubectl syntax, describes the command operations, and provides common examples. For details about each command, including all the supported flags and subcommands, see the [kubectl](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) reference documentation.

For installation instructions see [installing kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

Check that `kubectl` is installed properly:
```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.2", GitCommit:"bb9ffb1654d4a729bb4cec18ff088eacc153c239", GitTreeState:"clean", BuildDate:"2018-08-07T23:17:28Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

## Configure kubectl to connect to your Kubernetes cluster

In the previous section, we showed you how to setup a local, standalone Kubernetes cluster on your laptop using MiniKube. Minikube will automatically create (or update) your `kubectl` configuration file (stored, by default, in `~/.kube/config`).

Below is what a kube config file created by MiniKube typically looks like:
```
$ cat ~/.kube/config 
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
```

Note: If your kube config file is stored in a non-default place, you can pass its location to `kubectl` like so:
```
$ kubectl --kubeconfig=/home/xtof/.kube/my_kube_config get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    7d        v1.10.0
```

Or, export the `KUBECONFIG` environment variables like so:
```
$ export KUBECONFIG=/home/xtof/.kube/my_kube_config
$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    7d        v1.10.0
```

## Check the kubectl configuration

Check that kubectl is properly configured by getting the cluster state:

```
$ kubectl cluster-info
```
If you see a URL response, kubectl is correctly configured to access your cluster.

If you see a message similar to the following, kubectl is not correctly configured or not able to connect to a Kubernetes cluster:
```
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```

For example, if you are intending to run a Kubernetes cluster on your laptop (locally), you will need a tool like minikube to be installed first and then re-run the commands stated above.

If kubectl cluster-info returns the url response but you can’t access your cluster, to check whether it is configured properly, use:
```
$ kubectl cluster-info dump
```

## Syntax

Use the following syntax to run kubectl commands from your terminal window:

```
kubectl [command] [TYPE] [NAME] [flags]
```
where `command`, `TYPE`, `NAME`, and `flags` are:

* `command`: Specifies the operation that you want to perform on one or more resources, for example `create`, `get`, `describe`, `delete`.

* `TYPE`: Specifies the resource type. Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms. For example, the following commands produce the same output:
```
$ kubectl get pod pod1
$ kubectl get pods pod1
$ kubectl get po pod1
```

* `NAME`: Specifies the name of the resource. Names are case-sensitive. If the name is omitted, details for all resources are displayed, for example `kubectl get pods`.

When performing an operation on multiple resources, you can specify each resource by type and name or specify one or more files:

* To specify resources by type and name:
  * To group resources if they are all the same type: `TYPE1 name1 name2 name<#>`.
  * Example: `kubectl get pod example-pod1 example-pod2`

* To specify multiple resource types individually: `TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name<#>`.
  * Example: `kubectl get pod/example-pod1 replicationcontroller/example-rc1`

* To specify resources with one or more files: `-f file1 -f file2 -f file<#>`
  * Use YAML rather than JSON since YAML tends to be more user-friendly, especially for configuration files.
  * Example: `kubectl get pod -f ./pod.yaml`

* `flags`: Specifies optional flags. For example, you can use the `-s` or `--server` flags to specify the address and port of the Kubernetes API server.
**Important**: Flags that you specify from the command line override default values and any corresponding environment variables.

If you need help, just run `kubectl help` from the terminal window.

You can also get help on each command with:
```
$ kubectl <command> --help
# E.g.,
$ kubectl get --help
```

Also, use `kubectl options` for a list of global command-line options (applies to all commands).

## Common operations


