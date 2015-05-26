---
layout: post
title: "闲话Docker"
date: 2013-11-21 07:50
comments: true
categories: 
---

Docker是一个轻量级的应用程序容器，先看官方定义：

{% blockquote %}

Docker is an open-source project to easily create lightweight, portable, self-sufficient containers from any application. The same container that a developer builds and tests on a laptop can run at scale, in production, on VMs, bare metal, OpenStack clusters.

{% endblockquote %}

在官网的一个视频中，Docker对自己有一个更为形象的比喻：集装箱。Docker就相当于是集装箱本身，而各种应用程序就是里面的货物。所以，就可以理解为无论有多少台机器，通过Docker可以保证给每台机器都是同样的集装箱，这样可以简化大规模部署的问题。

Docker中定义了Image的概念，其实，可以简单的理解为一个container就是一个image，它实现了对image类似git的管理方式，因此，可以通过增量的方式改变当前image的内容，当客户端需要更新时，只需要pull下来最新的版本即可。

Docker官网有一个类似模拟命令行的小应用，可以让你体验一下，地址：http://www.docker.io/gettingstarted/#

但是，Docker官方还不建议在生产环境中使用Docker，原因是它还处在剧烈的开发中，很可能不同版本变化巨大，同时它还有一些依赖，如：

    1. kernel的版本最好是3.8及以上，因为docker基于LXC，而3.8版本下的LXC貌似有一些bug
    2. AUFS文件系统的支持

另外，现在Docker被编译为64bit的应用，如果要在32bit的机器上跑起来就要做一些额外的工作，有两种方式可以尝试：

    1. 装一些64位的运行库在32位的机器上
    2. 重新用32位的toolchain编译docker

如果在Mac上通过vagrant操作docker是一件十分方便的事情，只需要几步:

####1. git同步Docker的源码

{% codeblock lang:bash %}
git clone https://github.com/dotcloud/docker.git
cd docker
{% endcodeblock %}

做这步主要是因为里面包喊你了Vagrantfile的文件

####2. 启动vagrant并ssh进入

{% codeblock lang:bash %}
vagrant up
vagrant ssh
{% endcodeblock %}

####3. 运行docker

{% codeblock lang:bash %}
sudo docker
{% endcodeblock %}

如果要在一台全新的ubuntu 12.04LTS 64bit上运行Docker，那么步骤多一点，命令总结如下：

{% codeblock lang:bash %}
sudo apt-get update
{% endcodeblock %}

升级kernel到3.8，因为12.04的kernel为3.2

{% codeblock lang:bash %}
sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring -> upgrade kernel to 3.8.0.33

sudo shutdown -r now

uname -a -> check whether the kernel is correct or not
{% endcodeblock %}

{% codeblock lang:bash %}
apt-cache search linux-headers-$(uname -r)
{% endcodeblock %}

{% codeblock lang:bash %}
sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -“
sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list”   
sudo apt-get update
{% endcodeblock %}

{% codeblock lang:bash %}
sudo apt-get install lxc-docker
{% endcodeblock %}