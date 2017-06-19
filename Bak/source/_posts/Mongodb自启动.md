title: Mongodb自启动
date: 2017-06-01
tags: mongodb
categories: 
   - 后端
   - mongodb
------

写这个的时候刚好是6月1号，在这先祝母校CUMT108岁生日快乐！好怀念在学校儿童节放假的日子啊。废话不多说，下面进入正题

Mongodb在正常关闭的状态下，只要加到开机启动下，服务器重启就可以正常重启。这个没有任何问题。

那么如果服务器突然断电，然后重启的话，这样子的话，因为不是正常关闭启动，mongodb的进程就会被锁定，这时候一般就要删去db下面的mongod.lock或者打开这个文件，知道PID之后，kill掉这个PID。然后重启就没有问题。

所以在知道重启的方法之后，其实写上一个脚本，在服务器启动的时候执行就行了。脚本具体如下

``` bash
#！/bin/bash
lockfile='/usr/mongodb/db/mongod.lock'
nowtime=$(date +%Y%m%d)
if [ -f "${lockfile}" ]
then
 rm -rf ${lockfile}
 echo "=====delete success ==${nowtime}===="
else
 echo "======no exist====${nowtime}===="
fi
```

然后就是把这个脚本授权，加到/etc/rc.local 里面

``` bash
chmond 777 /usr/mongodb/delete.sh
/usr/mongodb/delete.sh 1>/usr/mongodb/logs/delete.file &
```

在我自己腾讯云的linux服务器测试没有问题，但是在公司的机器上测试遇到了一个问题

公司的机器上的rc.local不会开机执行，rc.local有下面的的内容

``` bash
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In constrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
```

翻译

``` bash
#这个文件是为了兼容性的问题而添加的。
#
#强烈建议创建自己的systemd服务或udev规则来在开机时运行脚本而不是使用这个文件。
#
#与以前的版本引导时的并行执行相比较，这个脚本将不会在其他所有的服务后执行。
#
#请记住，你必须执行“chmod +x /etc/rc.d/rc.local”来确保确保这个脚本在引导时执行。
```

然后按照说明的内容执行

``` bash
chmod +x /etc/rc.d/rc.local
```

然后重启测试，没有问题