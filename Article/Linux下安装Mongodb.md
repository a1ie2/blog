title: Linux下安装Mongodb
date:  2017-05-22
tags: mongodb
categories: 
   - 后端
   - mongodb   
------

网上有很多关于Mongodb的教程，因为项目用到Mongodb，所以记录一下自己安装使用Mongodb踩过的坑

## 安装步骤 ##

### 安装 ###

#### 下载安装包，把安装包下载到/usr/mongodb/下，方便以后安装 ####

``` bash
wget -P /usr/mongodb/ https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.4.tgz
```

#### 解压下载的包 ####

``` bash
tar -xvzf /usr/mongodb/mongodb-linux-x86_64-3.4.4.tgz
```

#### 重命名文件夹 ####

``` bash
mv mongodb-linux-x86_64-3.4.4.tgz mongodbserver
```

#### 创建配置文件 ####

``` bash
vim mongodb.conf
dbpath=/usr/mongodb/db
logpath=/usr/mongodb/logs/log.log
port=27017
fork=true
nohttpinterface=true
```

#### 把mongodb加入系统路径变量中，不用输入路径直接启动 ####

``` bash
export PATH=/usr/local/server/mongodb/bin:$PATH
```

查看是否成功

``` bash
echo $PATH
```

#### 把mongodb服务加到开机启动中 ####

``` bash
vi /ect/rc.local
/usr/mongodb/mongodbserver/bin/mongod --config /usr/mongodb/mongodb.conf
```

#### 错误解决 #### 

有时候会出现

``` bash
ERROR：child proecess failed ,excited with error number 1
```

网上提供的解决方法：

1. 关闭防火墙，或者打开端口，打开端口命令，检查机器是否ping的通

``` bash
firewall-cmd --zone=public --add-port=27017/tcp --permanent
```

2. 可以看看/usr/lib64/libssl.so.10 在不在，没有的话可以下一个，在的话，排除

3. 一般是权限问题，sudo启动就好了，具体操作如下

``` bash
sudo /usr/momngodb/mongodbserver/bin/mongod -f /usr/mongodb/mongodb.conf
```

4. 查看你的dbpath和logpath的文件夹是否存在(我遇到的问题)，不存在的话，创建一下，然后重新启动

#### mongodb的数据自动备份 #### 

因为怕有时候有特殊情况的出现，所以准备写一个自动备份和删除旧数据的脚本，放在linux的自启动任务下执行，具体流程如下

##### 创建自动备份脚本 #####

``` bash
#！/bin/bash
set fileformat=unix
sourcepath='/usr/mongodb/mongodbserver/'bin
targetpath='/usr/mongodb/mongoback/'
nowtime=$(date +%Y%m%d)
start()
{
 ${sourcepath}/mongodump --host 127.0.0.1 --port 27017 --out ${targetpath}/${nowtime}
}
execute()
{
 start
 if [ $? -eq 0 ]
 then 
  echo "back successfully"
 else
  echo "back failure"
 fi
}
if [ ! -d "${targetpath}/${nowtime}/" ]
then
 mkdir ${targetpath}/${nowtime}
fi
execute
echo "==============back end ${nowtime}==========="
```

增加脚本的权限

``` bash
chmod 777 /usr/mongodb/backmongo.sh
```

执行脚本，验证脚本是否有错

``` bash
./backmongo.sh
```

如果打出你的日志的话，说明没有错

接下来就是删除旧数据的脚本

``` bash
#！/bin/bash
targetpath='/usr/mongodb/mongoback'
nowtime=$(date -d '-7 days' "%Y%m%d")
if [ -d "${targetpath}/${nowtime}/" ]
then 
 rm -rf "${targetpath}/${nowtime}/"
 echo "======${targetpath}/${nowtime}/===delete ok==="
fi
echo "======${nowtime}=========="
```

同样要增加权限，验证是否有错

增加到定时任务里


``` bash
crontab -e 

30 2 * * * /usr/mongodb/backmongo.sh  1>/usr/mongodb/logs/backmongo.file &
0 2 * * *  /usr/mongodb/deletemongo.sh 1>/usr/mongodb/logs/deletemongo.file &

```

表示2点30 执行备份操作，2点执行删除操作，测试的时候可以把时间调一下测试，看看日志有没有数据到你指定的文件中，有没有备份成功

- [mongodb配置文件说明](http://document-1253712676.costj.myqcloud.com/mongo%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E.txt)