# Docker networking

This section provides an overview of Docker's default networking behavior, including the type of networks created by default and how to create your own user-defined networks. It also describes the resources required to create networks on a single host or across a cluster of hosts.

## Default networks

When you install Docker, it creates three networks automatically. You can list these networks using the docker network ls command:

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
90b3c15ef581        bridge              bridge              local
928f5811ef02        host                host                local
7e4ac9f86df1        none                null                local
```

These three networks are built into Docker. When you run a container, you can use the --network flag to specify which networks your container should connect to.

The `bridge` network represents the `docker0` network present in all Docker installations. Unless you specify otherwise with the `docker run --network=<NETWORK>` option, the Docker daemon connects containers to this network by default. You can see this bridge as part of a host's network stack by using the `ip addr show` command (or short form, `ip a`) on the host. (Note: the `ifconfig` command is deprecated. It may also work or give you a `command not found` error, depending on your system.)

```
$ ip addr show docker0
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:75:70:13 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c0ff:fe75:7013/64 scope link 
       valid_lft forever preferred_lft forever
#~OR~
$ ifconfig docker0
docker0   Link encap:Ethernet  HWaddr 02:42:c0:75:70:13  
          inet addr:172.17.0.1 Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:c0ff:fe75:7013/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:420654 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1162975 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:85851647 (85.8 MB)  TX bytes:1196235716 (1.1 GB)

$ docker network inspect bridge | jq '.[] | .IPAM.Config[].Subnet'
"172.17.0.0/16"
```
So, the usable range of IP addresses in our 172.17.0.0/16 subnet is: 172.17.0.1 - 172.17.255.254

The `none` network adds a container to a container-specific network stack. That container lacks a network interface.

The `host` network adds a container on the host's network stack. As far as the network is concerned, there is no isolation between the host machine and the container. For instance, if you run a container that runs a web server on port 80 using host networking, the web server is available on port 80 of the host machine.

The `none` and `host` networks are not directly configurable in Docker. However, you can configure the default `bridge` network, as well as your own user-defined bridge networks.

### The default bridge network

The default bridge network is present on all Docker hosts. If you do not specify a different network, new containers are automatically connected to the default bridge network.

The `docker network inspect` command returns information about a network:

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "90b3c15ef58175a11b023d735543284d5422a6405168d966280451694acdbe08",
        "Created": "2018-08-26T17:55:03.854812577-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

Run the following two commands to start two busybox containers, which are each connected to the default bridge network:
```
$ docker run -itd --name=container1 busybox
c5c8b37f47e451ac30f1a7e97bf6dafef55f1989efd2552e5daf0a4ec4c64c5a

$ docker run -itd --name=container2 busybox
5b711cc4d369d08ea03131a6c4ab994affb791a3604e705e016fad567a98046c
```

Inspect the bridge network again after starting two containers. Both of the busybox containers are connected to the network. Make note of their IP addresses, which will be different on your host machine than in the example below.
```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "90b3c15ef58175a11b023d735543284d5422a6405168d966280451694acdbe08",
        "Created": "2018-08-26T17:55:03.854812577-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "5b711cc4d369d08ea03131a6c4ab994affb791a3604e705e016fad567a98046c": {
                "Name": "container2",
                "EndpointID": "fe5c36b4496c202528063f727e7c870692645049cd6b27529115c9a607270889",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "c5c8b37f47e451ac30f1a7e97bf6dafef55f1989efd2552e5daf0a4ec4c64c5a": {
                "Name": "container1",
                "EndpointID": "ed3f068645e757f79231f57b51c3fc397c95aaec22d41375cf4b508ca92d8755",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

Containers connected to the default bridge network can communicate with each other by IP address.

You can `attach` to a running `container` to see how the network looks from inside the container. You are connected as `root`, so your command prompt is a `#` character.
```
$ docker attach container1

/ # ip -4 addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

From inside the container, use the ping command to test the network connection to the IP address of the other container (`container2`):
```
/ # ping -c3 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.112 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.107 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.110 ms

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.107/0.109/0.112 ms
```

Use the `cat` command to view the `/etc/hosts` file on the container. This shows the hostnames and IP addresses the container recognizes.
```
/ # cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	c5c8b37f47e4
```

To detach from the `container1` container and leave it running, use the keyboard sequence **CTRL-p CTRL-q**. If you wish, attach to `container2` and repeat the commands above.

The default `docker0` bridge network supports the use of port mapping and `docker run --link` to allow communications among containers in the `docker0` network. This approach is *not* recommended. Where possible, you should use user-defined bridge networks instead (see below).

* Get information on the network used by your running containers:

```
$ docker ps -q | wc -l
#~OR~
$ docker container ls --format '{{.Names}}' | wc -l
4  # => 4 running containers
$ docker network inspect bridge | jq '.[] | .Containers[].IPv4Address'
"172.17.0.2/16"
"172.17.0.5/16"
"172.17.0.4/16"
"172.17.0.3/16"
```
The output from the last command are the IP addresses of the 4 containers currently running on my host.

## Custom networks

* Create a Docker network
```
$ man docker-network-create  # for details
$ docker network create --subnet 10.1.0.0/16 --gateway 10.1.0.1 --ip-range=10.1.4.0/24 \
    --driver=bridge --label=host4network br04
```

* Use the above network with a given container:
```
$ docker run -it --name net-test --net br04 centos:latest /bin/bash
```

* Assign a static IP to a given container in the above (user created) network:
```
$ docker run -it --name net-test --net br04 --ip 10.1.4.100 centos:latest /bin/bash
```

Note: You can *only* assign static IPs to user created networks (i.e., you *cannot* assign them to the default "<code>bridge</code>" network).
