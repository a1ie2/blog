title: Zabbix使用笔记(二)：安装Agent端
date: 2017-06-22
tags: zabbix
categories: 
   - 后端
   - zabbix   
------

上一篇介绍安装Server端,这篇介绍安装Agent端，然后在监控页面上配置Agent端安装的主机就可以监控安装Agent端的主机了。


### Linux下安装Agent端 ### 

安装zabbix-agent


``` bash
rpm -ivh http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm

yum install -y zabbix-agent 
```

修改配置，启动服务


``` bash
vim /etc/zabbix/zabbix_agentd.conf 

Server=修改你的server所在的IP
#用于被动模式，数据获取，主动模式和被动模式按照自己需求选择。
ServerActive=修改你的server所在的IP
#用于主动模式，数据提交，主动模式和被动模式按照自己需求选择。

systemctl start zabbix-agent
```

> 关于主动和被动模式：
>  
> agent主动发送数据给server就叫主动模式
>
> server主动向agent要数据就是被动模式
>
> 如果你的agent很多而且还是主动模式，那你的server的性能就大打折扣，因为只有一个server得向很多的agent去要信息，可想而知！所以有时候还是可以去配置agent的被动模式


在监控页面配置主机

后台管理页面-->配置-->创建主机

![](http://images-1253712676.costj.myqcloud.com/zabbix/%E4%BA%8C/linux%E5%AE%89%E8%A3%85agent.png)

然后记得给你创建的主机添加模板，因为agent安装在Linux上的，所以选择的时候就直接选择Linux的模板`Template OS Linux`就行了

![](http://images-1253712676.costj.myqcloud.com/zabbix/%E4%BA%8C/linux%E5%AE%89%E8%A3%85agent2.png)

完成之后会自动跳到首页，等过个几分钟刷新一下，可以发现已启用，ZBX显示绿色

![](http://images-1253712676.costj.myqcloud.com/zabbix/%E4%BA%8C/linux%E5%AE%89%E8%A3%85agent3.png)


### Windows下安装Agent端 ###

官网下载windows下需要的文件，一定要下载和你server版本相同的文件，解压，拷贝对应32位还是64位的文件，我的机器是64位，所以拷贝出了下面的文件

![](http://images-1253712676.costj.myqcloud.com/zabbix/%E4%BA%8C/windows%E4%B8%8B%E5%AE%89%E8%A3%85agent1.png)

修改zabbix_agentd.win.conf里面的配置信息

- LogFile=你自己想存的地方，该文件夹一定要存在
- Server=你server安装的IP
- ServerActive=你server安装的IP
- Hostname=你自己想叫啥就叫啥，但是一定要记得，因为配置的时候是需要的

修改好之后，安装启动服务即可


``` bash
zabbix_agentd.exe --config <你自己的配置文件的绝对路径>  -i  

zabbix_agentd.exe --config <你自己的配置文件的绝对路径>  -s
```

到这，windows下的agent就算安装好了，然后就是在你的监控的页面配上你的agent,和linux上面配置的一样，**注意的是，名称天上你刚刚修改配置文件里面的hostname就行了**



至此，linux下的agent和windows下的agent已经配置安装成功，监控页面已经可以显示监控的信息了。