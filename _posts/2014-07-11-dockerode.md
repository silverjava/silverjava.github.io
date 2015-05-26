---
layout: post
title: "在Node中通过Dockerode操作Docker"
date: 2014-07-11 22:06
comments: true
categories: 
---
Docker作为轻量级的虚拟化技术已经发展到了1.1的版本，然而从版本0.3.4开始，它就已经开始提供一套 Remote API。这套API是完全基于RESTful方式组织和设计的，通过它，常规的Docker操作都可以完成，例如：可以通过发送POST请求到/container/create创建一个Container，通过发送GET请求到/container/{id}/json对某个Container执行inspect操作等等。这就为我们提供一种可以远程操作Docker的方式。

当前，基于这套API的基于不同语言的客户端已经有很多了，大家可以访问这里找到他们：[Remote API client libraries](https://docs.docker.com/reference/api/remote_api_client_libraries/)。

这篇文章要介绍的是Dockerode，它是基于Javascript的一个客户端，项目地址：https://github.com/apocas/dockerode 。在Node使用它非常简单，首先使用NPM安装：


``` bash install dockerode
npm install --save dockerode
```

下面的代码可以列出所有Image的信息：

``` javascript list all images

var Docker = require('dockerode');
var docker = new Docker({ socketPath: '/var/run/docker.sock' });

docker.listImages({}, function (err, data) {
  console.log(data);
});

```
在我本机得到的输出是：

``` javascript images
[ { Created: 1404876476,
    Id: 'c13376ac176f3de146ef5dc895fac2135e322b746070ee8c368f206b2e636ab3',
    ParentId: '84773e31a0d1fd7a0498fd2d4cf6d6f16c756c030f0a7ddcf1077cb653f4d3bb',
    RepoTags: [ 'sshd:latest' ],
    Size: 0,
    VirtualSize: 314749357 },
  { Created: 1404164147,
    Id: '58faa899733f1db4bf5722b12da74e6edf3c67c8f6d8db6559f547f9416f3c7e',
    ParentId: '6c3df001ea12dcf848ff51930954e2129ac8f5717ce98819237d2d5d3e8ddd25',
    RepoTags: [ 'ubuntu:14.10', 'ubuntu:utopic' ],
    Size: 0,
    VirtualSize: 195975166 },
  { Created: 1403128455,
    Id: 'c5881f11ded97fd2252adf93268114329e985624c5d7bb86e439a36109d1124e',
    ParentId: '5796a7edb16bffa3408e0f00b1b8dc0fa4651ac88b68eee5a01b088bedb9c54a',
    RepoTags: [ 'ubuntu:quantal' ],
    Size: 70975627,
    VirtualSize: 172064416 },
  { Created: 1403128361,
    Id: 'e54ca5efa2e962582a223ca9810f7f1b62ea9b5c3975d14a5da79d3bf6020f37',
    ParentId: '6c37f792ddacad573016e6aea7fc9fb377127b4767ce6104c9f869314a12041e',
    RepoTags: [ 'ubuntu:14.04', 'ubuntu:latest', 'ubuntu:trusty' ],
    Size: 8,
    VirtualSize: 276100357 } ]
```

其他例子就不再说了，大家有兴趣可以看看官方提供的例子。不过，这里有一点需要特别指出就是Dockerode对Stream支持的很好。有了它，我们就可以实现很多好玩的东西，比如：实现在Web页面上的Terminal，这个在我后续的文章会介绍如何实现，这里先给一个命令行版本：

``` javascript
var Docker = require('../lib/docker');
var fs     = require('fs');

var socket = process.env.DOCKER_SOCKET || '/var/run/docker.sock';
var stats  = fs.statSync(socket);

if (!stats.isSocket()) {
  throw new Error("Are you sure the docker is running?");
}

var docker = new Docker({ socketPath: socket });
var optsc = {
  'Hostname': '',
  'User': '',
  'AttachStdin': true,
  'AttachStdout': true,
  'AttachStderr': true,
  'Tty': true,
  'OpenStdin': true,
  'StdinOnce': false,
  'Env': null,
  'Cmd': ['bash'],
  'Dns': ['8.8.8.8', '8.8.4.4'],
  'Image': 'ubuntu',
  'Volumes': {},
  'VolumesFrom': ''
};

function handler(err, container) {
  var attach_opts = {stream: true, stdin: true, stdout: true, stderr: true};

  container.attach(attach_opts, function handler(err, stream) {
    // Show outputs
    stream.pipe(process.stdout);

    // Connect stdin
    var isRaw = process.isRaw;
    process.stdin.resume();
    process.stdin.setRawMode(true);
    process.stdin.pipe(stream);

    container.start(function(err, data) {
      // Resize tty
      var resize = function() {
        var dimensions = {
          h: process.stdout.rows,
          w: process.stderr.columns
        };

        if (dimensions.h != 0 && dimensions.w != 0) {
          container.resize(dimensions, function() {});
        }
      };
      
      resize();
      process.stdout.on('resize', resize);

      container.wait(function(err, data) {
        process.stdout.removeListener('resize', resize);
        process.stdin.removeAllListeners();
        process.stdin.setRawMode(isRaw);
        process.stdin.resume();
        stream.end();
        process.exit();
      });
    });
  });
}

docker.createContainer(optsc, handler);
```

这个例子也是官方提供的，有了它，我们就可以实现像连接SSH那样来操作一个Docker Container了。
