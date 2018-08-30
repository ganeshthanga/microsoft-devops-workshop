## Docker install

This document will show you how to install Docker on Linux (either Debian-based or Red Hat-based). See the end of the document for links on how to install Docker on macos and Windows.

### Debian-based distros

*Note: For this install, I will be using Ubuntu 16.04 LTS (Xenial Xerus). Docker requires a 64-bit version of Ubuntu as well as a kernel version equal to or greater than 3.10. My system satisfies both requirements.*

* Setup the docker repo to install from:
```
$ sudo apt-get update -y
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
$ echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
$ sudo apt-get update -y
```

Make sure you are about to install from the Docker repo instead of the default Ubuntu 16.04 repo:
```
$ apt-cache policy docker-engine
```

The output of the above command show look something like the following:
```
docker-engine:
  Installed: (none)
  Candidate: 17.05.0~ce-0~ubuntu-xenial
  Version table:
     17.05.0~ce-0~ubuntu-xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     17.04.0~ce-0~ubuntu-xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
...
```

* Install docker:
```
$ sudo apt-get install -y docker-engine
```

### Red Hat-based distros

*Note: For this install, I will be using CentOS 7 (release 7.2.1511). Docker requires a 64-bit version of CentOS as well as a kernel version equal to or greater than 3.10. My system satisfies both requirements.*

* Install Docker (the fast way):
```
$ sudo yum update -y
$ curl -fsSL https://get.docker.com/ | sh
```

* Install Docker (via a yum repo):
```
$ sudo yum update -y
$ sudo pip install docker-py
$ cat << EOF > /etc/yum.repos.d/docker.repo
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF

$ sudo rpm -vv --import https://yum.dockerproject.org/gpg
$ sudo yum update -y
$ sudo yum install docker-engine -y
```

### Post-installation steps

*Note: The following steps should be run on either your Debian-based or Red Hat-based distros.*

* Check on the status of docker:
```
$ sudo systemctl status docker

● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2018-08-22 22:44:10 UTC; 21s ago
     Docs: https://docs.docker.com
 Main PID: 3922 (dockerd)
   CGroup: /system.slice/docker.service
           ├─3922 /usr/bin/dockerd -H fd://
           └─3927 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim dock
```

* Make sure the docker service automatically starts after a machine reboot:
```
$ sudo systemctl enable docker
```

* Execute docker without `sudo`:
```
$ sudo usermod -aG docker $(whoami)
```
Log out and log back in to use docker without `sudo`.

* Check version of Docker installed:
```
$ docker version
Client:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:10:54 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:10:54 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

* Check that docker has been successfully installed and configured:
```
$ docker run hello-world
...
This message shows that your installation appears to be working correctly.
...
```

That's it! You should now have Docker successfully installed.

# External links

* [Install Docker on macos](https://docs.docker.com/docker-for-mac/install/#install-and-run-docker-for-mac)
* [Install Docker on Windows](https://docs.docker.com/docker-for-windows/install/#start-docker-for-windows)
