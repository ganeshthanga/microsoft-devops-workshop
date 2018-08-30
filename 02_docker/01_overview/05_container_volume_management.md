# Container volume managment

```
$ docker run -it --name voltest -v /mydata centos:latest /bin/bash

[root@bffdcb88c485 /]# df -h
Filesystem                   Size  Used Avail Use% Mounted on
none                         213G  173G   30G  86% /
tmpfs                        7.8G     0  7.8G   0% /dev
tmpfs                        7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/ubuntu--vg-root  213G  173G   30G  86% /mydata
shm                           64M     0   64M   0% /dev/shm
tmpfs                        7.8G     0  7.8G   0% /sys/firmware

[root@bffdcb88c485 /]# echo "testing" >/mydata/mytext.txt

$ docker inspect voltest | jq -crM '.[] | .Mounts[].Source'
/var/lib/docker/volumes/2a53fd295595690200a63def8a333b54682174923339130d560fb77ecbe41a3b/_data

$ sudo cat /var/lib/docker/volumes/2a53fd295595690200a63def8a333b54682174923339130d560fb77ecbe41a3b/_data/mytext.txt
testing

$ sudo /bin/bash -c \
  "echo 'this is from the host OS' >/var/lib/docker/volumes/2a53fd295595690200a63def8a333b54682174923339130d560fb77ecbe41a3b/_data/host.txt"

[root@bffdcb88c485 /]# cat /mydata/host.txt 
this is from the host OS
```

* Cleanup
```
$ docker rm voltest
$ docker volume ls
$ docker volume rm 2a53fd295595690200a63def8a333b54682174923339130d560fb77ecbe41a3b
```

* Mount host's current working directory inside container:
```
$ echo "my config" >my.conf
$ echo "my message" >message.txt
$ echo "aerwr3adf" >app.bin
$ chmod +x app.bin
$ docker run -it --name voltest -v ${PWD}:/mydata centos:latest /bin/bash

[root@f5f34ccb54fb /]# ls -l /mydata/
total 24
-rwxrwxr-x 1 1000 1000 10 Mar  8 19:29 app.bin
-rw-rw-r-- 1 1000 1000 11 Mar  8 19:29 message.txt
-rw-rw-r-- 1 1000 1000 10 Mar  8 19:29 my.conf
[root@f5f34ccb54fb /]# touch /mydata/foobar

$ ls -l ${PWD}
total 24
-rwxrwxr-x 1 bob   bob   10 Mar  8 11:29 app.bin
-rw-r--r-- 1 root  root   0 Mar  8 11:36 foobar
-rw-rw-r-- 1 bob   bob   11 Mar  8 11:29 message.txt
-rw-rw-r-- 1 bob   bob   10 Mar  8 11:29 my.conf

$ docker rm voltest
```
