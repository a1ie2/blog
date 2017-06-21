title: Zabbix使用笔记(一)：安装Server端
date:  2017-06-21
tags: zabbix
categories: 
   - 后端
   - zabbix   
------

安装zabbix环境：

1. 腾讯云的裸机centos
2. 防火墙关闭

### 创建用户和用户组 ###

zabbix官方文档表示，如果你用`root`去安装运行zabbix,他会自动切换到`zabbix`这个用户下面，所以，`zabbix`这个用户必须存在

具体可以看官方文档(反正我的英文烂到家)

> For all of the Zabbix daemon processes, an unprivileged user is required. If a Zabbix daemon is started from an unprivileged user account, it will run as that user

>However, if a daemon is started from a 'root' account, it will switch to a 'zabbix' user account, which must be present. To create such a user account (in its own group, “zabbix”) on Linux systems

所以先创建一个用户组，再创建一个用户

``` bash
groupadd zabbix
useradd -g zabbix zabbix
```


### 安装mysql ###

因为zabbix的数据都是存储在数据库中，zabbix支持了很多种数据库，但是因为我稍微熟悉MySQL(关键官方文档里面的也是用的MySQL)，所以这里就安装MariaDB这个数据库

``` bash
yum -y install mariadb-server mariadb-devel
```

安装完成之后开启，并且进入mysql

``` bash
systemctl start mariadb.service

mysql
```

这里要创建zabbix专用的数据库,并且把数据库的权限赋值给我们之前创建的用户

``` bash
create database zabbix character set utf8 collate utf8_bin;

grant all privileges on zabbix.* to zabbix@localhost identified by '<password>';
```

这里`<passpord>`换成你自己想设置的密码就行了

然后刷新一下，退出mysql即可,记得开启服务

``` bash
flush privileges;

exit
```

### 安装zabbix ###


下载zabbix的包，并且安装


``` bash
mkdir /usr/zabbix

wget P /usr/zabbix/  http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm

rpm -ivh  /usr/zabbix/zabbix-release-3.2-1.el7.noarch.rpm
```

至此，安装rpm包完成

现在安装zabbix端，因为上面我们使用了mysql，所以在这里我只安装server端的话，就要安装`zabbix-server-mysql`和`zabbix-web-mysql`


``` bash
yum install -y zabbix-server-mysql zabbix-web-mysql
```

安装完成之后，需要把zabbix的数据库初始化脚本导入到mysql里面

``` bash
zcat /usr/share/doc/zabbix-server-mysql-3.2.*/create.sql.gz | mysql -uzabbix -p zabbix
```

提示要输入密码，输入上面创建zabbix数据库的时候用的密码即可

### 修改server配置 ###

修改zabbix_server.conf，保证下面信息要有

- DBHost=localhost
- DBName=zabbix
- DBUser=zabbix
- DBPassword=<password>(这是你之前创建的时候用的密码)

``` bash
vi /etc/zabbix/zabbix_server.conf
```

开启server的服务

``` bash
systemctl start zabbix-server
systemctl enable zabbix-server
```

### 修改前端的时区配置 ###

``` bash
vi  /etc/httpd/conf.d/zabbix.conf
```

把`# php_value date.timezone Europe/Riga`改成`php_value date.timezone Asia/Shanghai`


如果SELinux的status是enabeld的话，设置 SELinux能连接到前端页面

开启httpd服务

``` bash
systemctl start httpd
```

页面打开 http//你的服务器ip/zabbix就可以了

然后一通设置，会跳转到登陆页面,默认的账号密码是：Admin,zabbix
