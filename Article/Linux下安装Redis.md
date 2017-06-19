title: Linux下安装Redis
date:  2017-05-17
tags: redis
categories: 
   - 后端
   - redis   
------

Redis在非关系型数据库中还是算蛮好用的，Windows安装很傻瓜，Linux安装其实也很傻瓜

### 安装步骤 ###

#### 下载Redis到指定目录 ####

``` bash
wget -P /usr/redis  http://download.redis.io/releases/redis-3.2.8.tar.gz
```

#### 解压下载的安装包 ####

``` bash
tar -zxvf redis-3.2.8.tar.gz
```

#### 定位到解压后的文件夹并且make ####

``` bash
cd redis-3.2.8/
make
```

这时候可能会出现下面的问题

![](http://images-1253712676.costj.myqcloud.com/Linux%E4%B8%8B%E5%AE%89%E8%A3%85Redis/redis2017051701.png)

这是因为gcc没有安装，得先安装gcc,然后才能make成功

``` bash
yum install gcc
```

安装完成之后，make又报了一个错

![](http://images-1253712676.costj.myqcloud.com/Linux%E4%B8%8B%E5%AE%89%E8%A3%85Redis/redis2017051702.png)

查看README里面有这样一段话

![](http://images-1253712676.costj.myqcloud.com/Linux%E4%B8%8B%E5%AE%89%E8%A3%85Redis/redis2017051703.png)

大概意思就是关于分配器allocator,如果有MALLOC这个环境变量的话，会用这个环境变量去创建Redis，而libc呢又不是默认的分配器，默认的是jemalloc，用jemalloc是因为比libc有更少的fragemnetation problems。但是现在jemalloc没有，所有就报错，make的时候加上MALLOC这个参数就行了

``` bash
make MALLOC=libc
```

完美解决

#### 复制启动redis所需要的文件 ####

定位到刚才make之后的src下，复制下面的文件到你自己指定的文件，其中Redis.conf不在src这个文件夹下，在一开始解压的文件夹下

``` bash
cp redis-server  /usr/redis
cp redis-benchmark /usr/redis
cp redis-cli  /usr/redis
cp redis.conf  /usr/redis
```

#### 启动redis ####

``` bash
cd /usr/redis/server
./redis-server redis.conf
```

#### 检验是否安装成功 ####

``` bash
./redis-cli
```

发现已经进入到本地的6379的端口了

![](http://images-1253712676.costj.myqcloud.com/Linux%E4%B8%8B%E5%AE%89%E8%A3%85Redis/redis2017051704.png)

随便set/get一个值

``` bash
set 1 1
get 1
1
```

检验没有问题

#### 关于redis.conf配置问题 #### 

redis的配置有很多参数，具体整理了一份，仅供参考，感谢提供文档的胡小宇大哥。

-  [redis.conf 参数说明](http://document-1253712676.costj.myqcloud.com/redis%E9%85%8D%E7%BD%AE%E9%83%A8%E7%BD%B2.doc)