# etcd
etcd is an open-source, distributed, key-value store, developed by CoreOS. etcd gracefully handles leader elections during network partitions and will tolerate machine failure, including the leader. These values can be watched, allowing your app to reconfigure itself when they change. This makes it a great solution for a system like Kubernetes, where things are event-driven by desired state, defined by submitting yaml/json objects to etcd.

It has a raft storage mechanism, running fewer nodes gives better performance, but sacrifices HA. You should aim to use the least ammount of nodes, with a minimum of 3 **OR** 5. The OR is important, because running 4 nodes, gives you very little advantage over running 3, in both cases, you can only tolerate 1 node failure. In order to achieve a two node failure tolerance, at least 5 nodes are necessary, and so on focusing on maintaing a "Quorum" or majority.

<Insert ETCD Quorum Visual Aid>

## Demo

**Note:** This is part of the Kubernetes "Control Plane" that is abstracted from the end-user for most managed platforms, such as AKS. This quick demonstration serves to help you understand the components, and the primary DevOps tasks associated.

### etcd3
The most recent version of etcd3 has been used by Kubernetes, since June 2016.

`export HostIP=$(ipconfig getifaddr en0)`

```
docker run \
  -d -p 2379:2379 -p 2380:2380 \
  --volume=$(pwd)/data:/etcd-data \
  --name etcd quay.io/coreos/etcd:latest \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data --name node1 \
  --initial-advertise-peer-urls http://${HostIP}:2380 --listen-peer-urls http://0.0.0.0:2380 \
  --advertise-client-urls http://${HostIP}:2379 --listen-client-urls http://0.0.0.0:2379 \
  --initial-cluster node1=http://${HostIP}:2380
```

Check the status of your etcd endpoint.

`docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] endpoint health`

`docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] put foo1 bar1`
`docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] put foo2 bar2`
`docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] put foo3 bar3`

```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] get \
  --prefix --limit=2 foo --print-value-only
```

```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] get foo1 \
  --print-value-only
```

Kubernetes would us a more path based storage, since you can watch the parent for new child objects.

```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] put /registry/services/specs/default/kubernetes '{ 
    "apiVersion": "v1",
    "kind": "Service",
    "metadata": {
        "labels": {
            "component": "apiserver",
            "provider": "kubernetes"
        },
        "name": "kubernetes",
        "namespace": "default"
    }
}'
```

```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] \
  get /registry/services/specs/default/kubernetes \
  --print-value-only
```

```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] watch \
  --prefix /registry/services/specs/default
```

*In Another Terminal, with `HostIP` properly set.*
```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] put /registry/services/specs/default/kubernetes2 '{ 
    "apiVersion": "v1",
    "kind": "Service",
    "metadata": {
        "labels": {
            "component": "apiserver",
            "provider": "kubernetes2"
        },
        "name": "kubernetes2",
        "namespace": "default"
    }
}'
```

*In the Previous Terminal You will see the `put` Trigger an Event*

*Output:*
```
PUT
/registry/services/specs/default/kubernetes2
{ 
    "apiVersion": "v1",
    "kind": "Service",
    "metadata": {
        "labels": {
            "component": "apiserver",
            "provider": "kubernetes2"
        },
        "name": "kubernetes2",
        "namespace": "default"
    }
}
```

It's worth mentioning that etcd also support time based leases, and keeps track of revisions. To avoid revisions piling up, Kubernetes compacts/removes these at a regular interval.

#### Back-Up
```
docker run -it --volume=$(pwd)/backup-data:/backup-data \
  -e ETCDCTL_API=3 tenstartups/etcdctl \
  --endpoints=[http://${HostIP}:2379] \
  snapshot save /backup-data/snapshot.db
```

#### Restore
First, to experiment with restore, you will want to destroy everything except our back-up data.
```
docker kill etcd
docker rm etcd
rm -Rf $(pwd)/data
```

```
docker run -it --volume=$(pwd)/data:/etcd-data \
  --volume=$(pwd)/backup-data:/backup-data \
  -e ETCDCTL_API=3 --entrypoint /bin/sh tenstartups/etcdctl -c "etcdctl \
  --endpoints=[http://${HostIP}:2379] \
  snapshot restore /backup-data/snapshot.db --name node1 \
  --initial-advertise-peer-urls http://${HostIP}:2380 \
  --initial-cluster node1=http://${HostIP}:2380 && mv /node1.etcd/* /etcd-data"
```

Note that the previous command, created a snap directory for the cluster to start from in the data directory. Bring up a fresh instance of etcd with that data directory's contents.

```
docker run \
  -d -p 2379:2379 -p 2380:2380 \
  --volume=$(pwd)/data:/etcd-data \
  --name etcd quay.io/coreos/etcd:latest \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data --name node1 \
  --initial-advertise-peer-urls http://${HostIP}:2380 --listen-peer-urls http://0.0.0.0:2380 \
  --advertise-client-urls http://${HostIP}:2379 --listen-client-urls http://0.0.0.0:2379 \
  --initial-cluster node1=http://${HostIP}:2380
```

```
docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] \
  get /registry/services/specs/default/kubernetes \
  --print-value-only
```

#### Authentication
etcd authenticates via certificate, both to add/remove data and for members joining the cluster. 

Generating Self-Signed Certs for etcd is outside the scope of this workshop, but the general premise is here, for setting up authentication backed etcd cluster.
https://github.com/coreos/etcd/tree/master/hack/tls-setup

### etcd2
Older deployments of Kubernetes may still use etcd2, and its important to know that breaking changes were made between version 2 and 3. There are viable upgrade paths for users that want to preserve cluster data, from legacy clusters, pre-etcd3. 

You will need to set the `HostIP` environment var, if not already set, prior to running the following commands, in a bash context. You should also run the steps for clean up, before bring up this version of etcd, in the same local environment.

`export HostIP=$(ipconfig getifaddr en0)`

```
docker run -v $(pwd)/data:/var/lib/etcd2 -d -p 4001:4001 -p 2380:2380 -p 2379:2379 \
 --name etcd quay.io/coreos/etcd:v2.3.8 \
 --data-dir=/var/lib/etcd2 \
 -name etcd0 \
 -advertise-client-urls http://${HostIP}:2379,http://${HostIP}:4001 \
 -listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
 -initial-advertise-peer-urls http://${HostIP}:2380 \
 -listen-peer-urls http://0.0.0.0:2380 \
 -initial-cluster-token etcd-cluster-1 \
 -initial-cluster etcd0=http://${HostIP}:2380 \
 -initial-cluster-state new
```

`docker run -it tenstartups/etcdctl -C http://${HostIP}:2379 member list` <We use bash and start an interactive session.>

### Clean Up

```
docker kill etcd
docker rm etcd
rm -Rf $(pwd)/data
```

## Additional Information

You can find additional info about docker etcd deployment options: https://coreos.com/etcd/docs/latest/v2/docker_guide.html