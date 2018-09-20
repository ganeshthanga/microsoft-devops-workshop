# AKS

Azure Provides a managed Kubernetes cluster service, that you can utilize to deploy your workloads.

In this section, we will walk through setting up an AKS cluster, which you can use to experiment in the coming sections and demonstrations.


## Pre-Reqs

1. You need to have access to an existing Azure subscription.
2. You will need to install the latest `azure-cli` tool. `azure-cli (2.0.45)`

## Steps

1. Log into the Azure Portal Dashboard
2. Click on "All services" in the top left corner.
	- ![image_01](aks_images/image_01.png)
3. Search for `aks` and select "Kubernetes services".
	- ![image_02](aks_images/image_02.png)
4. Click the `Add` button to create a new Cluster.
	- ![image_03](aks_images/image_03.png)
5. Fill in appropriate information for the Basic Tab
	- ![image_04](aks_images/image_04.png)
	- ![image_05](aks_images/image_05.png)
6. Fill in the appropriate information for the Authentication Tab
	- ![image_06](aks_images/image_06.png)
6. Fill in the appropriate information for the Networking Tab
	- ![image_07](aks_images/image_07.png)
6. Fill in the appropriate information for the Monitoring Tab
	- ![image_08](aks_images/image_08.png)
7. Add any tags, you feel may be appropriate in the Tags Tab.
8. Click "Review + Create" Tab and then Click "Create"
	- ![image_09](aks_images/image_09.png)
9. This can take about 30-45 minutes to complete, and you should be able to see your cluster.
	- ![image_10](aks_images/image_10.png)
10. Click on your cluster, and in the Overview Tab, click "View Kubernetes Dashboard".
11. In your command line, type `az login` and follow the instructions.
12. Run `az aks install-cli` to install `kubectl` into your local environment.
13. Run `az aks get-credentials` with your flags to set the kubectl context.


## Validate

After performing the steps above, you should be able to run the following command, and see some Pods running.

`kubectl get pods --all-namespaces`

## Enabling SSH

We will follow the azure docs for getting ssh access.

https://docs.microsoft.com/en-us/azure/aks/ssh

## Install Latest Version of Docker (Required for Automated Builds in Section 4)
Once you are onto a worker node, run the following commands to upgrade docker to `17.05.*`

You will need to do this on every node, in your cluster.

```
$ sudo vi /etc/apt/preferences.d/docker.pref
# Contents of the above file should look like this:
Package: docker-engine
Pin: version 17.05.*
Pin-Priority: 550

$ sudo apt-get update
$ sudo apt-get install -y docker-engine
```

## Continuing

The next sections wll explain kubectl, and give you a chance to flex some of these actions against your AKS cluster.
