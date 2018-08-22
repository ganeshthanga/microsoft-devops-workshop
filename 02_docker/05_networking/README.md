# Docker networking

## Default networks

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

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
bf831059febc        bridge              bridge              local
266f6df5c44e        host                host                local
ce79e4043a20        none                null                local
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

Note: You can ''only'' assign static IPs to user created networks (i.e., you ''cannot'' assign them to the default "bridge" network).
