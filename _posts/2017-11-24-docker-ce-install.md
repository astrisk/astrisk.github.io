---
title:  "Install Docker CE in CentOS 7"
date:   2017-11-24 18:37:00
categories: DOCKER
---

**Uninstall old docker**

{% highlight c %}

$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine

{% endhighlight %}

**Install using the repository**

{% highlight c %}

$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

{% endhighlight %}


**Optional**

{% highlight c %}

$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
$ sudo yum-config-manager --disable docker-ce-edge

{% endhighlight %}

**Install Docker CE**

{% highlight c %}

$ sudo yum install -y docker-ce-17.06.1.ce

{% endhighlight %}

**Start docker**

{% highlight c %}

$ sudo systemctl start docker

{% endhighlight %}

**Verify**

{% highlight c %}

$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9a0669468bf7: Pull complete
Digest: sha256:cf2f6d004a59f7c18ec89df311cf0f6a1c714ec924eebcbfdd759a669b90e711
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:

{% endhighlight %}

**Modify Images Path**

{% highlight c %}

$ mkdir /home/docker（你想要docker存放image的目录）

$ systemctl stop docker

$ vi /usr/lib/systemd/system/docker.service


[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/docker daemon -g /home/docker -H fd://
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target



$ cp -R /var/lib/docker/* /home/docker/

$ systemctl daemon-reload

$ systemctl start docker

{% endhighlight %}