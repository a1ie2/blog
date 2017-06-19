title: Linux下安装RabbitMQ
date:  2017-05-16
tags: rabbitmq
categories: 
   - 后端
   - rabbitmq   
------

之前项目要用到RabbitMQ,但是一直没有在Liunx下安装，正好趁着要用顺便动手装了一下,结果踩了不少坑。
最后发现真的超级简单的

### 安装步骤 ###

#### 安装 ####

首先百度了一下网上的安装教程，因为RabbitMQ都需要依赖于Erlang,所以我也是开始长路漫漫的安装Erlang,还遇到了各种少包的问题，当然后来才发现，自己真特么的傻！废话不多说，傻瓜式安装正式开始

##### 下载官网提供的包 #####

``` bash
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.9/rabbitmq-server-3.6.9-1.el7.noarch.rpm
```

我是下载的最新的包
因为我有强迫症，所以一定要找到下载的包在哪，并且移动到我自己安装的文件夹下，如果无所谓的话，可以忽略下面的步骤，但是一定要知道包在哪，要不然怎么装？！

##### 找到下载的包 #####

``` bash
whereis rabbitmq-server-3.6.9-1.el7.noarch.rpm
```

##### 移动包到自己的文件夹下 ##### 

``` bash
mv /proc/3207/cwd/rabbitmq-server-3.6.9-1.el7.noarch.rpm /usr/rabbitmq/
```

##### 安装下载的包 ##### 

关键的就是这步，原来一直用rpm来安装包，因为是先安装Erlang的，所以得先下Erlang的包，然后各种巴拉巴拉的包要下要装，所以后来想到了用yum安装不久行了嘛？！

``` bash
yum install /usr/rabbitmq/rabbitmq/rabbitmq-server-3.6.9-1.el7.noarch.rpm
```

中途会遇到需要确认的，直接”y”就行

![](http://images-1253712676.costj.myqcloud.com/Linux%E4%B8%8B%E5%AE%89%E8%A3%85RabbitMQ/rabbitmq2017051601.png)

然后就看到”Complete！”,这就安装完成了

##### 配置 ##### 

安装完成之后还需要配置，打开服务，打开界面管理

###### 找到包含rabbitmq-server.service的文件夹 ######

``` bash
whereis rabbitmq
```

然后自己去找

###### 打开管理界面，之后可以用15672端口在浏览器中查看信息 ######

``` bash
rabbitmq-plugin enable rabbitmq-management
```

###### 把rabbitmq添加到启动项 ######

``` bash
systemctl enable rabbitmq-server.service
```

之后就可用下面的命令启动服务了

``` bash
systemctl start rabbitmq-server.service
```

###### 测试登陆 ######

这时候就可以在页面上登陆查看了，默认guest只能在localhost下登陆，如果用IP或者域名查看的话，得先添加用户

http://***.***.***.***:15672/#(记得提前把端口打开)

如果用默认的账号和密码登陆不了的话，可以添加用户 具体操作如下
打开到rabbitmq安装文件夹下的sbin文件夹
然后执行下面的语句

``` bash
rabbitmqctl add_user admin admin
```

创建了一个账号是admin，密码是admin的用户。再把这个用户设置成管理员

``` bash
rabbitmqctl set_user_tags admin administrator
```

然后给amdin的用户开权限

``` bash
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

然后就可以登陆了

![](http://images-1253712676.costj.myqcloud.com/Linux%E4%B8%8B%E5%AE%89%E8%A3%85RabbitMQ/rabbitmq2017051602.png)

至此，完美安装。具体升入的研究rabbitmq就另说




