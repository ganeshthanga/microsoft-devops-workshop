# etcd
etcd is an open-source, distributed, key-value store, developed by CoreOS. etcd gracefully handles leader elections during network partitions and will tolerate machine failure, including the leader. These values can be watched, allowing your app to reconfigure itself when they change. This makes it a great solution for a system like Kubernetes, where things are event-driven by desired state, defined by submitting yaml/json objects to etcd.

## Demo

### etcd2

You will need to set the `HostIP` environment var, prior to running the following commands, in a bash context.

`export HostIP=10.0.0.63`

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

#### Clean Up

```
docker kill etcd
docker rm etcd
rm -Rf $(pwd)/data/*
```

### etcd3

`export HostIP=10.0.0.63`

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

`docker run -it -e ETCDCTL_API=3 tenstartups/etcdctl --endpoints=[http://${HostIP}:2379] endpoint health`

#### Clean Up

```
docker kill etcd
docker rm etcd
rm -Rf $(pwd)/data/*
```

## Additional Information

You can find additional info about docker etcd deployment options: https://coreos.com/etcd/docs/latest/v2/docker_guide.html