# Volume management

On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers. First, when a container crashes, kubelet will restart it, but the files will be lost (i.e., the container starts with a clean state). Second, when running containers together in a Pod it is often necessary to share files between those containers. The Kubernetes ''[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)'' abstraction solves both of these problems. A Volume is essentially a directory backed by a storage medium. The storage medium and its content are determined by the Volume Type.

In Kubernetes, a Volume is attached to a Pod and shared among the containers of that Pod. The Volume has the same life span as the Pod, and it outlives the containers of the Pod &mdash; this allows data to be preserved across container restarts.

Kubernetes resolves the problem of persistent storage with the Persistent Volume subsystem, which provides APIs for users and administrators to manage and consume storage. To manage the Volume, it uses the PersistentVolume (PV) API resource type, and to consume it, it uses the PersistentVolumeClaim (PVC) API resource type.

<dl>
  <dt>[PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes) (PV)</dt>
  <dd>a piece of storage in the cluster that has been provisioned by an administrator. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.</dd>

  <dt>[PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (PVC)</dt>
  <dd>a request for storage by a user. It is similar to a pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Persistent Volume Claims can request specific size and access modes (e.g., can be mounted once read/write or many times read-only).</dd>
</dl>

A Persistent Volume is a network-attached storage in the cluster, which is provisioned by the administrator.

Persistent Volumes can be provisioned statically by the administrator, or dynamically, based on the StorageClass resource. A StorageClass contains pre-defined provisioners and parameters to create a Persistent Volume.

A PersistentVolumeClaim (PVC) is a request for storage by a user. Users request Persistent Volume resources based on size, access modes, etc. Once a suitable Persistent Volume is found, it is bound to a Persistent Volume Claim. After a successful bind, the Persistent Volume Claim resource can be used in a Pod. Once a user finishes its work, the attached Persistent Volumes can be released. The underlying Persistent Volumes can then be reclaimed and recycled for future usage. See [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) for details.

* Access Modes
  * Each of the following access modes ''must'' be supported by storage resource provider (e.g., NFS, AWS EBS, etc.) if they are to be used.
  * ReadWriteOnce (RWO) &mdash; volume can be mounted as read/write by one node only.
  * ReadOnlyMany (ROX) &mdash; volume can be mounted read-only by many nodes.
  * ReadWriteMany (RWX) &mdash; volume can be mounted read/write by many nodes.
A volume can only be mounted using one access mode at a time, regardless of the modes that are supported.

## Example #1 - Using Host Volumes

As an example of how to use volumes, we can modify our previous "webserver" Deployment (see above) to look like the following:

```
$ cat webserver.yml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: webserver
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: hostvol
          mountPath: /usr/share/nginx/html
      volumes:
      - name: hostvol
        hostPath:
          path: /home/docker/vol
```

And use the same Service:
```
$ cat webserver-svc.yml

apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: webserver
```

Then create the deployment and service:
```
$ kubectl create -f webserver.yml
$ kubectl create -f webserver-svc.yml
```

Then, SSH into the webserver and run the following commands:
```
$ minikube ssh
minikube> mkdir -p /home/docker/vol
minikube> echo "Christoph testing" > /home/docker/vol/index.html
minikube> exit
```

Get the webserver IP and port:
```
$ minikube ip
192.168.99.100
$ kubectl get svc/web-service -o json | jq '.spec.ports[].nodePort'
32610

#~OR~

$ minikube service web-service --url

http://192.168.99.100:32610

$ curl http://192.168.99.100:32610
Christoph testing
```

## Example #2 - Using NFS

* First, create a server to host your NFS server (e.g., `sudo apt-get install -y nfs-kernel-server`).
* On your NFS server, do the following:
```
$ mkdir -p /var/nfs/general
$ cat << EOF >>/etc/exports
/var/nfs/general  10.100.1.2(rw,sync,no_subtree_check) 10.100.1.3(rw,sync,no_subtree_check) 10.100.1.4(rw,sync,no_subtree_check)
EOF
```
where the `10.x` IPs are the private IPs of your k8s nodes (both Master and Worker nodes).
* Make sure to install `nfs-common` on each of the k8s nodes that will be connecting to the NFS server.

Now, on the k8s Master node, create a Persistent Volume (PV) and Persistent Volume Claim (PVC):

* Create a Persistent Volume (PV):
```
$ cat << EOF >pv.yml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /var/nfs/general
    server: 10.100.1.10  # NFS Server's private IP
    readOnly: false
EOF

$ kubectl create --validate -f pv.yml
$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
mypv      1Gi        RWX            Recycle          Available
```

* Create a Persistent Volume Claim (PVC):
```
$ cat << EOF >pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
EOF

$ kubectl create --validate -f pvc.yml
$ kubectl get pvc
NAME      STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc   Bound     mypv      1Gi        RWX

$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM             STORAGECLASS   REASON    AGE
mypv      1Gi        RWX            Recycle          Bound     default/nfs-pvc                            11m
```

* Create a Pod:
```
$ cat << EOF >nfs-pod.yml 

apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod
  labels:
    name: nfs-pod
spec:
  containers:
  - name: nfs-ctn
    image: busybox
    command:
      - sleep
      - "3600"
    volumeMounts:
    - name: nfsvol
      mountPath: /tmp
  restartPolicy: Always
  securityContext:
    fsGroup: 65534
    runAsUser: 65534
  volumes:
    - name: nfsvol
      persistentVolumeClaim:
        claimName: nfs-pvc
EOF

$ kubectl create --validate -f nfs-pod.yml
$ kubectl get pods -o wide
NAME     READY     STATUS    RESTARTS   AGE       IP            NODE
busybox  1/1       Running   9          2d        10.244.2.22   k8s.worker01.local
```

* Get a shell from the `nfs-pod` Pod:
```
$ kubectl exec -it nfs-pod -- sh
/ $ df -h
Filesystem                Size      Used Available Use% Mounted on
172.31.119.58:/var/nfs/general
                         19.3G      1.8G     17.5G   9% /tmp
...
/ $ touch /tmp/this-is-from-the-pod
```

* On the NFS server:
```
$ ls -l /var/nfs/general/
total 0
-rw-r--r-- 1 nobody nogroup 0 Jan 18 23:32 this-is-from-the-pod
```

It works!

