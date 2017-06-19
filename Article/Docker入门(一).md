title: Docker入门(一)
date: 2017-06-06
tags: docker
categories: 
   - 后端
   - docker   
------

很久之前就听说Docker的大名，但是一直没有去研究。公司现在的持续交付系统就是用的Docker,但是看不到源码，加上手上的项目用到mongo,借此机会，就想初步的看一下docker到底是什么

网上关于docker的文章很多很多，我也看了很多，但是看归看，自己实践起来又是另一件事了

首先你要知道以下几点

1. docker到底是什么
2. docker里面的镜像是啥
3. docker里面的容器是啥

就我的理解

1. docker就是一个虚拟机，只不过很快，有点很多优点
2. 镜像呢，类似于一个虚拟机的环境，没镜像你这个虚拟机怎么运行呢？？
3. 容器呢就是一个镜像的实例，一个镜像可以运行很多个实例，但是你改了一个实例并不影响另一个实例，基于一个镜像的实例都是隔离的

其实，当你不明白这些概念的时候，也不要紧，自己动动手就知道是啥了。说实话，我一开始也不明白，但是自己动过手，就大概有个初步的理解。因为网上有很多教程，但是不要跟着别人的教程一步一步走，因为，这样很傻！哈哈哈

> 下面的教程是基于centos 7 的

## 安装docker ##


``` bash
yum -y install docker
```

安装完成，并没有启动,所以要启动一下

``` bash
systemctl start docker.service
```

## docker的基本操作 ##

### 查看docker的镜像 ###

``` bash
docker images
```

![](http://images-1253712676.costj.myqcloud.com/docker/docker01/docker2071060601.png)

上面是我已经pull了centos的一个镜像

pull镜像命令

``` bash
docker pull centos:latest
```

后面不加:latest他会默认拉去最新的，这个latest就是tag,标签。创建自己的镜像的时候可以设置，不设置的话，默认的就是latest

### 创建自己想要的镜像 ###

镜像是不能修改的，容器是在镜像基础上加上一层可写层。因为我们要运行一个自己的基于mongo的实例。所以，执行下面的命令

``` bash
docker run -i -t centos:latest /bin/bash
```

-t:让Docker分配一个伪终端并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开。

这时候我们就进入到了一个centos系统的容器中了，一定要记住root@f5ab9d924617的f5ab9d924617，这个后面得用

![](http://images-1253712676.costj.myqcloud.com/docker/docker01/docker2071060602.png)

然后你想干嘛就干嘛，比如你要安装mongo并启动，可以参考我之前的mongo安装

### 保存自己的容器成为新的镜像 ###

``` bash
docker commit f5ab9d924617 xiao5/mongo
```

保存之后就可以通过docker images 查看自己刚刚创建的镜像了

![](http://images-1253712676.costj.myqcloud.com/docker/docker01/docker2071060603.png)

### 运行我们自己创建的镜像的一个容器 ###

``` bash
docker run -p 27017:27017 -idt xiao5/mongodb /usr/mongodb/delete.sh
```

-d 以守护态运行

-p 27017:27017 把容器的27017端口映射到主机的27017端口

因为我是写一个脚本运行的，当你在你的镜像中安装mongo时，把mongo的bin文件夹加入到环境变量里面的话，可以把/usr/mongodb/delete.sh替换成mongod

这时候可以通过命令查看运行的容器，发现成功运行

![](http://images-1253712676.costj.myqcloud.com/docker/docker01/docker2071060604.png)

然后测试看看能不能连的上

![](http://images-1253712676.costj.myqcloud.com/docker/docker01/docker2071060605.png)

### 停止运行的容器 ###

``` bash
docker stop <collectionid>
```

或者

``` bash
docker kill <collectionid>
```

stop 命令呢是让容器有处理时间，保存程序执行现场，然后退出

kill 强干，快速退出


### 删除容器和镜像 ###

容器停止了，并不代表没有了

``` bash
docker ps -a
```

上面的命令表示查看所有的容器，包括停止的容器

删除全部容器

``` bash
docker rm $(docker ps -aq)
```

删除某一个容器

``` bash
docker rm <collectionid>
```

删除某一个镜像

``` bash
docker rmi <imageid>
```

删除全部镜像

``` bash
docker rmi $(docker images -q)
```

至此docker算是初级入门，因为我也是刚刚入门，所以很多东西都不知道说的对不对。docker算是一个很强大的工具，很值得去深入