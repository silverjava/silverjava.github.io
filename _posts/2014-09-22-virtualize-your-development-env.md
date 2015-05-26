---
layout: post
title: "为你的应用添加虚拟化环境"
date: 2014-09-22 22:29
comments: true
categories: 
---

当作为程序员的你加入一个新项目，又或是因为兴趣爱好在github发现一个有趣的项目时，通常我们第一件需要做的事情就是检查自己的环境是不是符合这个应用的需要，因为我们希望在本地先把它运行起来。做的好的应用会为你提供非常完备的脚本帮你完成所有准备环境需要的工作，但是，有更多的应用通常是提供类似tutorial的说明文档，你需要照着它一步步的做下去，如果中间出了问题，那么，你会看看有没有FAQ环节，如果有就祈祷希望在这个环节可以Cover你的问题。不过，通常不幸的事情总会发生，接着我们可能会因为环境问题而耗费大量的时间。而当准备环境变成一个团队都需要做的事情的时候，这个成本就可想而知了。

我最近在做一个小应用程序的时候就碰到了类似的问题。这个应用是一个基于Web的终端程序（Web-Based Terminal）。它的功能很简单：你在浏览器的下拉列表中选择任意你喜欢的Linux操作系统，然后就可以启动一个基于该操作系统的终端，接着就可以像使用真实终端一样在里面敲一些命令行。例如：当你选择了ubuntu 14.04，接着就可以像使用它的终端一样在浏览器中操作这个系统。

这个应用虽然小，但是技术栈也不算简单。它是基于NodeJS的应用，服务器端使用了Express，客户端当然是AngularJS。同时为了支持可以选择不同的Linux操作系统，我使用了Docker作为容器。而前后端的通信则使用了Web-Socket（Socket-IO）。就是这样一个技术栈的应用，配置起来也并不简单。首先，需要安装NodeJS，然后是该应用使用的包，然而像Socket-IO这样的库还存在需要重新编译的可能。接着，就需要安装Docker。考虑到不同的操作系统，安装方式也可能是千差万别的。另外，如果中间出了错，排错的过程也会是非常耗时的。借助虚拟化技术可以很轻易的解决这个问题。我将以上述这个应用为例说明如何为应用添加虚拟化环境。

首先，安装虚拟化必要的软件，我相信这些软件对于很多程序员来说都不陌生：

	- VirtualBox：https://www.virtualbox.org/ 
	- Vagrant：http://www.vagrantup.com/
	- Ansible: http://www.ansible.com/

安装完成后，测试所需软件是不是都已经安装好了：

	➜  ~  vagrant -v
	Vagrant 1.6.3
	
	➜  ~  ansible --version
	ansible 1.3.4

接着，进入该应用的根目录，并创建**Vagrantfile**:

	➜  ~  cd your/app/base/directory
	➜  ~  vagrant init ubuntu/trusty64

执行完这步，在应用的根目录会生成**Vagrantfile**文件，**Vagrantfile**是**Vagrant**用来描述虚拟机的文件，而命令**vagrant init ubuntu/trusty64**是创建一个使用ubuntu 14.04(trusty)的虚拟机。

如果查看当前**Vagrantfile**，它大概是这样（我已经将所有注释删除了）：

	➜  docker-app git:(master) cat Vagrantfile
	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
	VAGRANTFILE_API_VERSION = "2"

	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  		config.vm.box = "ubuntu/trusty64"
	end

如果这个时候运行**vagrant up**命令，**Vagrant**会在**vagrantcloud**上搜索名为**ubuntu/trusty64**的box镜像，并将它下载到本地。接着用这个镜像文件启动虚拟机。当启动完成后，你应该可以执行**vagrant ssh**登录进这个全新的ubuntu 14.04的虚拟机：

	➜  ~  vagrant up
	Bringing machine 'default' up with 'virtualbox' provider...
	==> default: Checking if box 'ubuntu/trusty64' is up to date...
	==> default: Clearing any previously set forwarded ports...
	==> default: Clearing any previously set network interfaces...
	==> default: Preparing network interfaces based on configuration...
    	default: Adapter 1: nat
	==> default: Forwarding ports...
    	default: 22 => 2222 (adapter 1)
	==> default: Running 'pre-boot' VM customizations...
	==> default: Booting VM...
	==> default: Waiting for machine to boot. This may take a few minutes...
    	default: SSH address: 127.0.0.1:2222
    	default: SSH username: vagrant
    	default: SSH auth method: private key
    	default: Warning: Connection timeout. Retrying...
	==> default: Machine booted and ready!
	==> default: Checking for guest additions in VM...
	==> default: Mounting shared folders...


	➜  ~  vagrant ssh
	Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-35-generic x86_64)

 	* Documentation:  https://help.ubuntu.com/

  	System information as of Mon Sep 22 15:17:26 UTC 2014

  	System load:  0.9               Processes:              97
  	Usage of /:   4.6% of 39.34GB   Users logged in:        0
  	Memory usage: 9%                IP address for eth0:    10.0.2.15
  	Swap usage:   0%                IP address for docker0: 172.17.42.1

  	Graph this data and manage this system at:
    	https://landscape.canonical.com/

  	Get cloud support with Ubuntu Advantage Cloud Guest:
    	http://www.ubuntu.com/business/services/cloud


	Last login: Mon Sep 22 08:25:57 2014 from 10.0.2.2
	vagrant@vagrant-ubuntu-trusty-64:~$

到这里，我们已经有一个干净的操作系统了，并且如果你通过**vagrant ssh**命令登录进虚拟机，并检查虚拟机中的**/vagrant**目录，你会发现**Vagrant**会把与**Vagrantfile**在同一级的所有文件**mount**到这个目录，换句话说，你可以在这个目录找到该项目的所有源文件。

接下来，就需要考虑如何**provision**这个干净的系统了。这个时候，我们回头看看这台机器到底需要哪些配置：

	- 安装Docker
	- 至少pull一个docker镜像，例如：docker pull centos:centos7
	- 安装NodeJS
	- 安装Global的NodeJS工具，例如：nodemon
	- 在应用中执行npm install安装所有服务端的依赖包
	- 在应用中执行bower install安装所有客户端需要的javascript库

以上这些是运行应用需要配置的工具及环境，然而，为了提升安装效率，我们通常还需要替换官方源为国内源并保证一些工具已经被安装了：

	- 使用国内ubuntu的源替代官方源
	- 保证必要的工具，如：vim，g++等，是已被安装的

我们先从简单的任务开始：替换官方源。这里，我们使用**Ansible**作为provision的工具，当然，**Vagrant**同样支持使用Chef，Puppet，Shell等方式。

首先创建并进入**ansible**目录：

	➜  ~  take ansible

按照**Ansible**官方推荐的最佳实践方式，创建第一个role **common**用来做比较通用的任务:

	➜  ~  mkdir -p common/tasks
	➜  ~  mkdir -p common/files
	➜  ~  touch common/tasks/main.yml

这里，我们创建了两个目录：**tasks**和**files**。通常，**tasks**存放任务的yml文件，而**files**目录则用来存放像源列表这样的静态文件。于是，我们将已经准备好的sources.list文件加入到**files**，并按照如下编辑main.yml文件：
	
	- name: replace source repo to speed up the download speed
  	  copy: src=sources.list dest=/etc/apt/sources.list
 
该**Ansible**的任务定义了用**files**目录中的**sources.list**文件替换虚拟机中**/etc/apt/sources.list**文件。

此时，我们完成了**common**这个role需要做的第一个任务。那么，如何执行呢？我们需要在刚才创建的**ansible**目录下创建一个新的文件site.yml并添加如下内容：

	---
	- hosts: all
  	  sudo: yes
  	  roles:
    	- common

同时，我们还需要修改**Vagrantfile**让**Vagrnat**知道相应**Ansible**脚本的位置，编辑**Vagrantfile**：

	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
	VAGRANTFILE_API_VERSION = "2"

	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  		config.vm.box = "ubuntu/trusty64"

  		config.vm.provision "ansible" do |ansible|
    		ansible.playbook = "ansible/site.yml"
  		end
	end

我们添加了一个新的代码块**config.vm.provision**，其中，我们指定了使用**ansible**作为provision的工具，并且指定了site.yml文件的路径。现在，我们可以测试第一个任务是不是可以正确工作了，回到项目根目录执行如下命令：

	➜  ~  vagrant provision

当执行完后，检查目录**/etc/apt/**下的sources.list文件是否已经被替换为我们提供的版本。

接下来，我们通过在**common**中的另外一个任务保证g++是已经被安装的，如果没有则安装。向**common**的**main.yml**中添加一个新任务：

	- name: install development tools
  	  apt: name=g++ state=present install_recommends=no update_cache=true

这个任务要求g++需要存在，如果没有则安装一个新的，其中**update_cached=true**意思是首先执行apt-get update。

接着再使用**vagrant provision**测试这个任务是否能被成功执行。

到现在为止，我们基本完成了对于role **common**的任务描述，下面我们将关注点转移到Docker身上。

同样的步骤，需要创建一个新的role **docker**：

	➜  ~  mkdir -p docker/tasks
	➜  ~  touch docker/tasks/main.yml

由于不需要有静态文件，所以这里我们不创建**files**目录。

对于**docker**，第一个任务必然是安装Docker，修改**main.yml**添加如下任务：

	- name: install docker
  	  shell: curl -sSL https://get.docker.io/ubuntu/ | sudo sh

为了测试，需要修改**site.yml**，将**docker**加入：

	---
	- hosts: all
  	  sudo: yes
  	  roles:
    	- common
    	- docker

继续运行**vagrant provision**并在虚拟机中运行以下命令检查Docker是否已经安装好了：

	vagrant@vagrant-ubuntu-trusty-64:~$ docker version
	Client version: 1.2.0
	Client API version: 1.14
	Go version (client): go1.3.1
	Git commit (client): fa7b24f
	OS/Arch (client): linux/amd64

对于role **docker**，我们还需要至少一个镜像，向**main.yml**添加新任务下载镜像：

	- name: pull images
  	  shell: docker pull {{item}}
  	  with_items:
    	- centos:centos6
    	- centos:centos7
    	- ubuntu:14.10

这里使用了Ansible提供的循环机制，这个任务会分别下载centos6，centos7以及ubuntu 14.10这三个镜像。再次执行**vagrant provision**，检查该任务是否正常工作（由于需要下载，这个任务可能会执行比较长的时间，因此建议只下载一个镜像即可）。

下面，我们将安装NodeJS。按照上述同样的方法创建role **node**，并向其**main.yml**中添加如下任务：

	- name: add nodejs ppa repo
  	  apt_repository: repo="ppa:chris-lea/node.js" update_cache=true

	- name: install nodejs
  	  apt: pkg=nodejs state=latest install_recommends=no

	- name: install global packages
  	  npm: name={{item}} global=yes state=latest
  	  with_items:
    	- nodemon
    	- bower

将**node**添加到**site.yml**中，并执行**vagrant provision**进行验证。

最后，添加role **app**，并为其**main.yml**中添加以下任务：

	- name: npm install all packages
  	  npm: path=/vagrant/app

	- name: bower install all client libs
  	  sudo: false
  	  command: chdir=/vagrant/app bower install

同样，将**app**加入**site.yml**，并测试。

最终，**ansible**目录结构如下：

	ansible
	├── roles
	│   ├── app
	│   │   └── tasks
	│   │       └── main.yml
	│   ├── common
	│   │   ├── files
	│   │   │   └── sources.list
	│   │   └── tasks
	│   │       └── main.yml
	│   ├── docker
	│   │   └── tasks
	│   │       └── main.yml
	│   └── node
	│       └── tasks
	│           └── main.yml
	└── site.yml

现在，我们可以使用如下命令对整个过程进行一次完整的测试：
	
	➜  ~  vagrant destroy
	➜  ~  vagrant up

执行成功后，在虚拟机内部应用程序需要环境都已经配置好了。当然，为了能够访问我们的web server，还需要做一点修改，将虚拟机内部的3000端口forward到主机的3000端口。修改**Vagrantfile**：

	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
	VAGRANTFILE_API_VERSION = "2"

	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  		config.vm.box = "ubuntu/trusty64"

  		config.vm.provision "ansible" do |ansible|
    		ansible.playbook = "ansible/site.yml"
  		end
  		
  		config.vm.network :forwarded_port, guest: 3000, host: 3000
	end

这样，当在虚拟机内部执行**nodemon app.js**后，我们就可以在主机浏览器中输入：localhost:3000 访问应用了。

通过为应用添加虚拟化环境，我们自动化了环境配置，这大大简化了配置环境的过程。而且该虚拟化环境是可以被随时销毁和重建的，因此，也并不用担心因为某些操作导致整个环境崩溃的问题。
