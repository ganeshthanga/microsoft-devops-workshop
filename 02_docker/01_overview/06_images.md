# Images

### Saving and loading images

```
$ docker pull centos:latest
$ docker run -it centos:latest /bin/bash

[root@29fad368048c /]# yum update -y
[root@29fad368048c /]# echo xtof >/root/built_by.txt
[root@29fad368048c /]# exit

$ docker commit reverent_elion centos:xtof
$ docker rm reverent_elion
$ docker images
REPOSITORY   TAG      IMAGE ID       CREATED         SIZE
centos       xtof     e0c8bd35ba50   3 seconds ago   463MB
centos       latest   980e0e4c79ec   1 minute ago    197MB

$ docker history centos:xtof
IMAGE          CREATED             CREATED BY                                      SIZE
e0c8bd35ba50   27 seconds ago      /bin/bash                                       266MB               
980e0e4c79ec   18 months ago       /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>      18 months ago       /bin/sh -c #(nop)  LABEL name=CentOS Base ...   0B                  
<missing>      18 months ago       /bin/sh -c #(nop) ADD file:e336b45186086f7...   197MB               
<missing>      18 months ago       /bin/sh -c #(nop)  MAINTAINER <nowiki>https://gith...</nowiki>   0B
```

* Save the original `centos:latest` image we pulled from Docker Hub:
```
$ docker save --output centos-latest.tar centos:latest
```

Note that the above command essentially tars up the contents of the image found in `/var/lib/docker/image` directory.

```
$ tar tvf centos-latest.tar 
-rw-r--r-- 0/0        2309 2016-09-06 14:10 980e0e4c79ec933406e467a296ce3b86685e6b42eed2f873745e6a91d718e37a.json
drwxr-xr-x 0/0           0 2016-09-06 14:10 ad96ed303040e4a7d1ee0596bb83db3175388259097dee50ac4aaae34e90c253/
-rw-r--r-- 0/0           3 2016-09-06 14:10 ad96ed303040e4a7d1ee0596bb83db3175388259097dee50ac4aaae34e90c253/VERSION
-rw-r--r-- 0/0        1391 2016-09-06 14:10 ad96ed303040e4a7d1ee0596bb83db3175388259097dee50ac4aaae34e90c253/json
-rw-r--r-- 0/0   204305920 2016-09-06 14:10 ad96ed303040e4a7d1ee0596bb83db3175388259097dee50ac4aaae34e90c253/layer.tar
-rw-r--r-- 0/0         202 1969-12-31 16:00 manifest.json
-rw-r--r-- 0/0          89 1969-12-31 16:00 repositories
```

* Save space by compressing the tar file:
```
$ gzip centos-latest.tar  # .tar -> 195M; .tar.gz -> 68M
```

* Delete the original `centos:latest` image:
```
$ docker rmi centos:latest
```

* Restore (or load) the image back to our local repository:
```
$ docker load --input centos-latest.tar.gz
```

## Tagging images

* List our current images:
```
$ docker images
REPOSITORY    TAG    IMAGE ID            CREATED             SIZE
centos        xtof   e0c8bd35ba50        About an hour ago   463MB
```

* Tag the above image:
```
$ docker tag e0c8bd35ba50 xtof/centos:v1
$ docker images
REPOSITORY    TAG    IMAGE ID            CREATED             SIZE
centos        xtof   e0c8bd35ba50        About an hour ago   463MB
xtof/centos   v1     e0c8bd35ba50        About an hour ago   463MB
```

Note that we did not create a new image, we just created a new tag of the same/original `centos:xtof` image.

Note: The maximum number of characters allowed in a tag is 128.
