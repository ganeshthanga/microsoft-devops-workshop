# Install Jenkins

Assuming helm is installed locally and on to your cluster, we can use that to deploy the stable jenkins chart. We will also setup ingress, and connect our jenkins deployment to the internet via ingress rules, in our chart.

`cd examples`

Lets take a moment to review `jenkins-values.minikube.yml` and/or `jenkins-values.aks.yml`

**Minikube** Exposed as NodePort
`helm upgrade --install jenkins stable/jenkins -f jenkins-values.minikube.yml`

**AKS** Exposed as LoadBalancer
`helm upgrade --install jenkins stable/jenkins -f jenkins-values.aks.yml`

If you experience issues, you can roll back by deleting the release.

`helm delete jenkins --purge`

## Post-Install

```
NOTES:
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT/login

3. Login with the password from step 1 and the username: admin

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine
#################################################################################
######   WARNING: Persistence is disabled!!! You will lose your data when   #####
######            the Jenkins pod is terminated.                            #####
#################################################################################
Configure the Kubernetes plugin in Jenkins to use the following Service Account name jenkins using the following steps:
  Create a Jenkins credential of type Kubernetes service account with service account name jenkins
  Under configure Jenkins -- Update the credentials config in the cloud section to use the service account credential you created in the step above.
```

```
examples$ printf $(kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
wN5068py7u
examples$ export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
examples$ export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
examples$ echo http://$NODE_IP:$NODE_PORT/login
http://10.0.2.15:31234/login
```

The password is correct, but the ip address this command prints out, was not accurate for my environment. I ran `minikube ssh`, then `ifconfig`. Try the addresses provided in the output, more than likely it will be `eth` related.

ifconfig
```
eth1      Link encap:Ethernet  HWaddr 08:00:27:93:C5:3A  
          inet addr:192.168.99.101  Bcast:192.168.99.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe93:c53a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2480 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1290 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:662160 (646.6 KiB)  TX bytes:1074965 (1.0 MiB)
```

That means, in my case I want to naviate to http://192.168.99.101:31234/login and use my admin username and password to log in.