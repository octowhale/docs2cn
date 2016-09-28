# 创建属于自己的镜像

> 2016-09-28  octowhale@github

[原文](https://docs.docker.com/engine/tutorials/dockerimages/)


docker images是容器的技术。每次使用 ` docker run ` 命令时都需要指定使用的镜像。之前的章节中使用的镜像都已经存在，例如 ` ubuntu 和 training/webapp `。

你也学习了如何在docker hub搜索并下载镜像到本地计算机。如果本机计算机上没有该镜像，则会从远程仓库中下载，默认仓库为[ Docker Hub Registry](https://hub.docker.com/)

在本节中，你将学习更多关于docker镜像的只是，包括：
+ 管理和使用本机镜像
+ 创建基础镜像
+ 上传镜像到Docker Hub Registry


## 查看本机镜像列表

使用 ` docker images ` 可以列出本地计算机上的所有镜像。

```bash
$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               1d073211c498        3 days ago          187.9 MB
busybox             latest              2c5ac3f849df        5 days ago          1.113 MB
training/webapp     latest              54bb4e8718e8        5 months ago        348.7 MB

```

你可以找到之前使用过的镜像。这些镜像都是之前在启动容器时从DockerHub上下载的。当你查看镜像列表时，需要注意三个重要信息：
+ 镜像所在的仓库，例如ubuntu
+ 镜像的标签(TAG)，例如14.04
+ 镜像的` IMAGE ID `

> **Tips**： 你可以使用第三方的[图形工具 dockviz tool](https://github.com/justone/dockviz)或[镜像网站](https://imagelayers.io/)查看镜像数据。


仓库中可能包含了一个镜像的不同衍生版，我们使用的ubuntu包含的衍生版有10.04,12.04,12.10,13.04,13.10和14.04。每个衍生版都有一个独立的tag。你可以通过tag指定需要使用的衍生版，例如：
```bash
ubuntu:14.04
```

因此当你的容器运行一个指定tag的镜像，命令如下：
```bash
$ docker run -t -i ubuntu:14.04 /bin/bash
```

如果你想使用12.04，则为：
```bash
$ docker run -t -i ubuntu:12.04 /bin/bash
```

如果命令中不指定tag，例如 ` ubuntu `， 那么docker默认使用 ` ubuntu:latest ` 镜像。

> **Tips**： 使用镜像时应该总是指定tag。这样你将明确知道正在使用的是什么衍生版，这将对排错很有帮助。


## 获得一个新镜像

如果你使用的镜像本机不存在，那么docker会自动下载。不过这样会增加部分启动时间。如果你需要预先下载一个镜像，可以使用 ` docker pull ` 命令。 加入你准备下载 ` centos ` 镜像：
```bash 
$ docker pull centos

Using default tag: latest
latest: Pulling from library/centos
f1b10cd84249: Pull complete
c852f6d61e65: Pull complete
7322fbe74aa5: Pull complete
Digest: sha256:90305c9112250c7e3746425477f1c4ef112b03b4abe78c612e092037bfecc3b7
Status: Downloaded newer image for centos:latest
```

你可以看到镜像的每个层(layer)。而且现在启动容器时不用在等待下载镜像了。
```bash
$ docker run -t -i centos /bin/bash
bash-4.1#
```


## 查找镜像
docker的其中一个特点在于人们基于不同目的创建了各种各样的镜像，并且很多人将他们的镜像上传到[Docker Hub](https://hub.docker.com/)。你可以访问Docker Hub搜索这些镜像。
![Docker_hub_search.png](https://docs.docker.com/engine/tutorials/search.png)

你可以在命令行界面使用 ` docker search ` 命令查找镜像。加入你想查找一个安装了Ruby和Sinatra的镜像，可以使用 ` docker search sinatra ` 命令查询包含了sinatra的所有镜像。
```bash
$ docker search sinatra
NAME                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
training/sinatra                       Sinatra training image                          0                    [OK]
marceldegraaf/sinatra                  Sinatra test app                                0
mattwarren/docker-sinatra-demo                                                         0                    [OK]
luisbebop/docker-sinatra-hello-world                                                   0                    [OK]
bmorearty/handson-sinatra              handson-ruby + Sinatra for Hands on with D...   0
subwiz/sinatra                                                                         0
bmorearty/sinatra                                                                      0
. . .
```

你可以看到命令返回了很多包含了sinatra关键字的镜像；列表中包含了镜像名称，描述和stars(受欢迎程度，如果一个用户喜欢该镜像则可以为镜像加star)；以及官方版本和自动创建状态。[官方仓库](https://docs.docker.com/docker-hub/official_repos)由docker.Inc管理一系列的docker仓库。 [自动创建](https://docs.docker.com/engine/tutorials/dockerrepos/#automated-builds)允许你去验证镜像的源和内容。

目前你见过了两种镜像仓库：
+ 一种为 ` ubuntu ` 这类官方镜像，被称为基础镜像或者根镜像。这些基础镜像是有Docker Inc提交与创建、验证、支持。基础镜像可以使用一个单词作为名称。
+ 另一种为 ` training/sinatra ` 这类用户镜像。这类镜像是docker用户自己创建和维护，你可以通过前缀辨别镜像的维护者，例如这里的 ` training `。


## 拉取镜像

这里我们使用 `training/sinatra`镜像。 你可以使用 ` docker pull ` 命令下载镜像
```bash
$ docker pull training/sinatra 
```

下载完成后，就可以在容器中应用该镜像了
```bash
$ docker run -t -i training/sinatra /bin/bash

root@a8cb6ce02d85:/#
```

## 创建你自己的镜像


