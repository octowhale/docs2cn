# 理解镜像,容器和存储驱动器

> 2016-10-09 octowhale@github

[ 原文 ](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)


为了有效的使用存储分区,你必须要理解docker是如何创建和存储的。之后，需要理解容器是如何使用这些镜像的。最后，你需要知道一些简单的镜像和容器的操作方法。


## 镜像和镜像层

每个docker镜像涉及了一系列表示文件系统差异的只读镜像层。这些镜像层相互堆叠在一起构成了容器的根文件系统。下面的图片展示了ubuntu 15.04是由4层镜像层堆叠的。
![image-layers.jpg](https://docs.docker.com/engine/userguide/storagedriver/images/image-layers.jpg)

docker 存储驱动器代表了这些所有堆叠在一起的镜像层，并提供一个统一的视图。

当你创建一个容器时，首先在底层堆栈的上方添加了一个全新的，thin，可写的层，该层通常被层为容器层。容器中的所有更改都发生在容器层中，例如创建、修改和删除文件。下面的图片展示了一个基于 ubuntu 15.04 的容器。

![container-layers.jpg](https://docs.docker.com/engine/userguide/storagedriver/images/container-layers.jpg)


## 内容寻址存储

docker 1.10 引入了一个全新的**内容寻址存储**模块。 这是一个用于在磁盘上定位镜像和层数据的完整的新方法。 在此之前， 镜像和层数据的引用和储存需要使用一个随机生成的UUID。 在新模块中，这将被替换成安全的*内容hash*。

该模块提高了安全性，提供了内建方法避免ID冲突，并且保证数据在push、pull、load和save等操作后的完整性。 这也实现了更好的共享镜像层，即使这些镜像层不是来至于相同的build。

下图展示了之前的图片的一个更新版本，提示了由docker 1.10执行的改变，
![container-layers-cas.jpg](https://docs.docker.com/engine/userguide/storagedriver/images/container-layers-cas.jpg)

可以看到，所有镜像层ID都是使用hash算法， 然而容器ID还是使用一个随机生成的UUID。

几个关于新模块的注意事项，
+ 移植已存在的镜像
+ 镜像和层文件系统结构

docker 1.10之前版本创建和pull的镜像，如果需要使用新模块需要进行移植。 移植操作会在更新docker daemon更新后自动执行，并会计算镜像的安全校验码。 在移植完成后， 所有镜像和tag会使用新的安全ID。

尽管移植过程自动且透明，但是计算量大。这意味着如果镜像数据较多，移植过程中会持续一段时间；同时docker daemon也不会相应其他请求。

移植工具允许你在升级docker daemon之前完成镜像的移植。这意味着docker daemon不需要自己完成镜像移植，因此避免了任何相关的维护时间。同时工具也提供了手动移植镜像的方法，这样就可以分发到其他已运行最新版本的docker daemon环境中。

移植工具作为一个容器，由docker.inc提供。下载地址：https://github.com/docker/v1.10-migrator/releases。

启动“migrator”镜像时需要将docker主机的数据目录暴露给容器。如果你使用的是默认docker数据路径，命令如下：
```bash
$ sudo docker run --rm -v /var/lib/docker:/var/lib/docker docker/v1.10-migrator
```

如果你使用 ` devicemapper ` 存储驱动器， 你需要使用选项 ` --privileged `，这样容器才能访问你的存储驱动器。


### 移植案例

下面的案例展示了移植工具在运行docker 1.9.1和使用AUFS存储驱动器的主机上使用。 docker主机配置为 AWS EC2 t2.micro, 1 vCPU, 1GB RAM 和一个 8GB的SSD EBS卷。 docker 数据目录[/var/lib/docker]使用了2GB的空间。
```
$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jenkins             latest              285c9f0f9d3d        17 hours ago        708.5 MB
mysql               latest              d39c3fa09ced        8 days ago          360.3 MB
mongo               latest              a74137af4532        13 days ago         317.4 MB
postgres            latest              9aae83d4127f        13 days ago         270.7 MB
redis               latest              8bccd73928d9        2 weeks ago         151.3 MB
centos              latest              c8a648134623        4 weeks ago         196.6 MB
ubuntu              15.04               c8be1ac8145a        7 weeks ago         131.3 MB

$ sudo du -hs /var/lib/docker

2.0G    /var/lib/docker

$ time docker run --rm -v /var/lib/docker:/var/lib/docker docker/v1.10-migrator

Unable to find image 'docker/v1.10-migrator:latest' locally
latest: Pulling from docker/v1.10-migrator
ed1f33c5883d: Pull complete
b3ca410aa2c1: Pull complete
2b9c6ed9099e: Pull complete
dce7e318b173: Pull complete
Digest: sha256:bd2b245d5d22dd94ec4a8417a9b81bb5e90b171031c6e216484db3fe300c2097
Status: Downloaded newer image for docker/v1.10-migrator:latest
time="2016-01-27T12:31:06Z" level=debug msg="Assembling tar data for 01e70da302a553ba13485ad020a0d77dbb47575a31c4f48221137bb08f45878d from /var/lib/docker/aufs/diff/01e70da302a553ba13485ad020a0d77dbb47575a31c4f48221137bb08f45878d"
time="2016-01-27T12:31:06Z" level=debug msg="Assembling tar data for 07ac220aeeef9febf1ac16a9d1a4eff7ef3c8cbf5ed0be6b6f4c35952ed7920d from /var/lib/docker/aufs/diff/07ac220aeeef9febf1ac16a9d1a4eff7ef3c8cbf5ed0be6b6f4c35952ed7920d"
<snip>
time="2016-01-27T12:32:00Z" level=debug msg="layer dbacfa057b30b1feaf15937c28bd8ca0d6c634fc311ccc35bd8d56d017595d5b took 10.80 seconds"

real    0m59.583s
user    0m0.046s
sys     0m0.008s
```

` time ` 命令估计了 ` docker run ` 命令执行了多长时间。 可以看到， 移植7个镜像总共消耗了大概一分钟； 其中包括pull ` docker/v1.10-migrator ` 镜像的时间（大约3.5s）。同样的操作，在m4.10xlarge EC2 instance with 40 vCPUs, 160GB RAM and an 8GB provisioned IOPS EBS volume 上，结果如下：
```
real    0m9.871s
user    0m0.094s
sys     0m0.021s
```

这说明了移植操作的效率受硬件影响。


## 容器和层

容器和镜像的最主要区别是顶部的可写层。容器的所有写入操作（添加/修改）都被保存在这个可写层。当容器被删除的时候，对应的可写层也被删除。但底层镜像保持不变。

由于每个容器拥有其对应的**可写容器层(thin writable container layer)**， 并且所有改变都被保存在这个容器层中，这意味着多个容器可以共享访问同一个底层镜像，并拥有独立的数据状态。 下入展示了多个容器共享同一个ubuntu 15.04镜像：
![sharing-layers.jpg](https://docs.docker.com/engine/userguide/storagedriver/images/sharing-layers.jpg)

docker存储驱动负责启用和管理镜像层和可写容器层。  How a storage driver accomplishes these can vary between drivers.  docker镜像和容器管理的两个关键技术为： 可堆栈镜像层`stackable image layers` 和 写入时备份`copy-on-write(CoW)`。


## copy-on-write 策略

共享是优化资源的好方式。

copy-on-write 是一个共享与复制的的策略。系统进程更需要相同的数据分享相同的实力数据，而非拥有他们自己的副本。 从某些角度来说，如果进程需要修改或写入数据，只需要为进程获取一份数据副本，且该进程有权限修改副本；其他进程则继续使用原始数据。

docker使用 copy-on-write 技术同时用于镜像和容器。 CoW策略优化了镜像磁盘空间使用和容器启动的性能。 下一节将着重于 copy-on-write 如何通过共享和复制在镜像和容器之间实现平衡的。


### Sharing promotes smaller images

所有镜像和容器层都存在于docker主机的*本地存储区域*，且通过存储驱动管理。 基于linux版的docker主机通常位于 ` /var/lib/docker/ ` 。

在使用 ` docker pull ` 和 ` docker push `时， docker客户端报告镜像层信息。下面的命令是pull ` ubuntu:15.04 ` 时的信息：
```
$ docker pull ubuntu:15.04

15.04: Pulling from library/ubuntu
1ba8ac955b97: Pull complete
f157c4e5ede7: Pull complete
0b7e98f84c4c: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:5e279a9df07990286cce22e1b0f5b0490629ca6d187698746ae5e28e604a640e
Status: Downloaded newer image for ubuntu:15.04
```

从输出中可以看到， 该命令实际上pull了4个镜像层。每一行都列出了镜像层的UUID或者hash加密。 ` ubuntu:15.04 ` 就是由这4层构成的。

每个镜像层都被保存在docker主机的本地存储内属于各自的目录中。

docker 1.10之前的版本中， 所有layer被保存在以自己的 ` image ID ` 命令的目录中。 然而，这种方式在 1.10以后并不适用了。 例如，下面的命令为docker 1.9.1版本时， pull下的镜像信息
```
$  docker pull ubuntu:15.04

15.04: Pulling from library/ubuntu
47984b517ca9: Pull complete
df6e891a3ea9: Pull complete
e65155041eed: Pull complete
c8be1ac8145a: Pull complete
Digest: sha256:5e279a9df07990286cce22e1b0f5b0490629ca6d187698746ae5e28e604a640e
Status: Downloaded newer image for ubuntu:15.04

$ ls /var/lib/docker/aufs/layers

47984b517ca9ca0312aced5c9698753ffa964c2015f2a5f18e5efa9848cf30e2
c8be1ac8145a6e59a55667f573883749ad66eaeef92b4df17e5ea1260e2d7356
df6e891a3ea9cdce2a388a2cf1b1711629557454fd120abd5be6d32329a0e0ac
e65155041eed7ec58dea78d90286048055ca75d41ea893c7246e794389ecf203
```

注意，4个目录名称可以与layer ID 匹配。 现在，对比 docker 1.10 版本的区别，
```
$ docker pull ubuntu:15.04
15.04: Pulling from library/ubuntu
1ba8ac955b97: Pull complete
f157c4e5ede7: Pull complete
0b7e98f84c4c: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:5e279a9df07990286cce22e1b0f5b0490629ca6d187698746ae5e28e604a640e
Status: Downloaded newer image for ubuntu:15.04

$ ls /var/lib/docker/aufs/layers/
1d6674ff835b10f76e354806e16b950f91a191d3b471236609ab13a930275e24
5dbb0cbe0148cf447b9464a358c1587be586058d9a4c9ce079320265e2bb94e7
bef7199f2ed8e86fa4ada1309cfad3089e0542fec8894690529e4c04a7ca2d73
ebf814eccfe98f2704660ca1d844e4348db3b5ccc637eb905d4818fbfb00a06a
```

可以看出， 4个目录名称与镜像的layer ID 不匹配。

无视 1.10 版本前后的镜像管理差别， docker所有版本仍然允许镜像层共享。比如说，如果你pull的一个镜像中某些镜像层已经在其他镜像中pull了，那么docker daemon会探测到，并置pull那些本地没有的镜像层。在第二次pull后，两个镜像可以共享公共镜像层。

为了阐释这一点， 使用刚才pull的 ` ubuntu:15.04 ` ， 随便更改一些东西，并在此基础上创建一个新的镜像。 其中一种方法则是使用 ` Dockerfile ` 和 ` docker build ` 命令：

1. In an empty directory, create a simple Dockerfile that starts with the ubuntu:15.04 image.
```
FROM ubuntu:15.04
```

2. Add a new file called “newfile” in the image’s /tmp directory with the text “Hello world” in it.
When you are done, the Dockerfile contains two lines:
```
 FROM ubuntu:15.04

 RUN echo "Hello world" > /tmp/newfile
```

3. Save and close the file.
4. From a terminal in the same folder as your Dockerfile, run the following command:
```
 $ docker build -t changed-ubuntu .

 Sending build context to Docker daemon 2.048 kB
 Step 1 : FROM ubuntu:15.04
  ---> 3f7bcee56709
 Step 2 : RUN echo "Hello world" > /tmp/newfile
  ---> Running in d14acd6fad4e
  ---> 94e6b7d2c720
 Removing intermediate container d14acd6fad4e
 Successfully built 94e6b7d2c720
```

> Note: The period (.) at the end of the above command is important. It tells the docker build command to use the current working directory as its build context.

5. Run the docker images command to verify the new changed-ubuntu image is in the Docker host’s local storage area.
```
 REPOSITORY       TAG      IMAGE ID       CREATED           SIZE
 changed-ubuntu   latest   03b964f68d06   33 seconds ago    131.4 MB
 ubuntu           15.04    013f3d01d247   6 weeks ago       131.3 MB
```

6. 使用 ` docker history ` 命令查看创建 ` changed-ubuntu ` 镜像使用了哪些镜像层。
```
 $ docker history changed-ubuntu
 IMAGE               CREATED              CREATED BY                                      SIZE        COMMENT
 94e6b7d2c720        2 minutes ago       /bin/sh -c echo "Hello world" > /tmp/newfile    12 B 
 3f7bcee56709        6 weeks ago         /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B  
 <missing>           6 weeks ago         /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.879 kB
 <missing>           6 weeks ago         /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   701 B
 <missing>           6 weeks ago         /bin/sh -c #(nop) ADD file:8e4943cd86e9b2ca13   131.3 MB
```

` docker history ` 命令的输出信息展示了新的 ` 94e6b7d2c720 ` 镜像层位于最上方。 该层是最新被添加的，因为是通过 ` echo "Hello world" > /tmp/newfile ` 命令创建的。 下面4个镜像层与组成 ` ubuntu:15.04 ` 的镜像层相同。

> **注意：**  由于docker 1.10引入的内容寻址存储模块， 镜像历史数据不再被保存在每个镜像层的config文件中。 现在所有镜像历史数据都是作为一个字符串保存在同一个config文件中。 这个可能会造成一些镜像层显示 ` "missing" ` 。 这种正常行为可以忽略。
>
> You may hear images like these referred to as flat images.

注意， ` changed-ubuntu ` 并不拥有属于自己的所有镜像层副本。 下图中我们可以看出， 新镜像共享使用了 ` ubuntu:15.04 ` 的底层镜像层：
![saving-space.jpg](https://docs.docker.com/engine/userguide/storagedriver/images/saving-space.jpg)

