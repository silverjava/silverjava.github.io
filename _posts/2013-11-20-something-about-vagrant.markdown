---
layout: post
title: "Vagrant二三事"
date: 2013-11-20 07:32
comments: true
categories: 
---

Vagrant简单来说就是一个轻量级的虚拟机，官方的定义是：

{% blockquote %}
Create and configure lightweight, reproducible, and portable development environments.
{% endblockquote %}

它可以帮助你在很短的时间内建立起一个或多个虚拟环境，就好像你同时在操作那么多台机器一样。这个特性在当今如此强调分布式计算的大环境下尤其有用。

我最近在项目中的工作是尝试搭建一个持续发布的环境，这个环境大致的方向是：可以自动化配置多态机器节点，同时还可以在不停机的情况发发布更新的应用程序。即使现在的硬件环境还没有准备，但借助vagrant就可以开始尝试部署这样的环境了。

这篇文章主要是对最近使用的一些命令和场景做一个总结（如何安装vagrant就不说了，下个安装包就行了）。

Vagrant有个依赖，就是需要本机安装VirtualBox或者VMware，由于vbox现在是免费的，所以建议安装它，这样整个环境就准备好了。

Vagrant是基于命令行的，所有操作都可以通过命令行来完成，要快速建立一个ubuntu 12.04LTS，其实很简单：

####1. 创建一个文件夹，随便什么名字

{% codeblock lang:bash %}
mkdir vagrant
{% endcodeblock %}

####2. 进入到目录

{% codeblock lang:bash %}
cd vagrant
{% endcodeblock %}

####3. 运行init命令

{% codeblock lang:bash %}
vagrant init
{% endcodeblock %}

这个命令会在这个目录生成Vagrantfile的配置文件，这个文件是ruby格式，里面有详细的关于如何配置虚拟机的说明，不过暂时还用不到它。

####4. 运行up命令

{% codeblock lang:bash %}
vagrant up
{% endcodeblock %}

这个命令会首先去vagrant的网站下载默认的box文件，例如：ubuntu 12.04LST 32bit是precise32.box。这个文件雷系虚拟机的镜像，之后的所有操作都是基于这个文件的。

####5. 运行ssh命令

{% codeblock lang:bash %}
vagrant ssh
{% endcodeblock %}

执行完后，会发现你已经在这个虚拟机中了，执行pwd可以查看当前目录。


接下来，我们看如何使他可以支持多台主机且可以相互通信。

打开Vagrantfile文件，在Vagrant.configure的block里面通过config.vm.box来定义主机名，例如定义一个host主机和一个slave主机

{% codeblock lang:ruby %}
config.vm.define "host", primary: true do |host|
      host.vm.box = "host"
end 

config.vm.define "slave" do |slave|
      slave.vm.box = “slave"
end 
{% endcodeblock %}

其中，对于host主机，我们还指定了primary：true，他的意思是如果单独执行vagrant ssh，这就是进入host主机，如果像进入slave主机，那么必须给ssh加个名字：vagrant ssh slave。

这时，在这个目录中执行vagrant up，它会尝试把这两台主机都启动起来。

现在两台主机配置好了，下一步就看如何可以使他们想会通信。这需要配置vm.network，分别给这两个主机的定义多加一行配置

{% codeblock lang:ruby %}
config.vm.define "host", primary: true do |host|
      host.vm.box = “host"

      host.vm.network "forwarded_port", guest: 80, host: 8080
      host.vm.network "private_network", ip: "192.168.1.10" 
end

config.vm.define "slave" do |slave|
      slave.vm.box = “slave”

      slave.vm.network "private_network", ip: "192.168.1.11" 
end 
{% endcodeblock %}

我们分别配置了host和slave的vm.network使用private_network，并指定IP地址。对于host主机还多加了一行端口映射。这样，如果在host装一个apache的http服务，在slave中就可以访问到了。
