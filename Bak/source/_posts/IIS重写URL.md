title: IIS重写URL
date: 2017-06-06
tags: iis
categories: 
   - 后端
   - iis  
------

### 需求 ###

外部URL定位到站点下的URL，比如，外部地址为http://xxx/news/1234,1231221.html，但是我的站点并没有类似结构和html

### 实现 ### 

#### 条件 ####

1. IIS7以及以上
2. URL重写模块(如果没有安装的话,可以下载安装，[点击下载](http://document-1253712676.costj.myqcloud.com/urlrewrite2.exe))
3. 会写正则表达式，当然不会写也可以百度

#### 步骤 #### 

##### 安装IIS的重写模块 #####

就上面的软件下载，安装就行了。安装完成之后，IIS所有的站点下会出现以下信息

![](http://images-1253712676.costj.myqcloud.com/IIS%E9%87%8D%E5%86%99URL/IIS%E9%87%8D%E5%86%99RUL.2017060601.png)

打开Web平台安装程序，找到产品，产品下找到服务器，右边列表找到对应的URL重写模块，点击添加即可

![](http://images-1253712676.costj.myqcloud.com/IIS%E9%87%8D%E5%86%99URL/IIS%E9%87%8D%E5%86%99RUL.2017060602.png)

添加完成之后，站点主页会有URL重写模块出现

![](http://images-1253712676.costj.myqcloud.com/IIS%E9%87%8D%E5%86%99URL/IIS%E9%87%8D%E5%86%99RUL.2017060603.png)

至此，所需要的都已经全部安装完成了

##### 写自己要的规则 #####

点击URL重写，点击入站规则的空白规则建立入站规则。入站规则就是把请求的URL解析成我们站点的URL,出战规则就是把我们站点的URL重写成你想要的URL。这里我们需求是别的URL解析成我们站点的URL。新建入展规则

![](http://images-1253712676.costj.myqcloud.com/IIS%E9%87%8D%E5%86%99URL/IIS%E9%87%8D%E5%86%99RUL.2017060604.png)

这里我们主要关注的是匹配URL，因为我们要把’/news/1234,1231221.html’转换成’pages/edu-detail.html?channelID=1234&newscode=1231221’

所以第一步，正则提取出要转化的URL里面的信息，就是匹配URL这个模块，点击匹配URL下的测试模式，因为由上面可知，我们需要1234和1231221这两个信息，正则找出这两个信息，这里要记住下图的{R:1},{R;2},下面有用

![](http://images-1253712676.costj.myqcloud.com/IIS%E9%87%8D%E5%86%99URL/IIS%E9%87%8D%E5%86%99RUL.2017060605.png)

可以之后，关闭，会提示是否保存，点击保存即可

##### 模块介绍 #####

###### 条件模块 ######

条件模块一般是匹配是不是该站点，比如我想要把abc.com/123,123.html转换成abc.com/id=123&id2=123，但是会有人盗链，会请求def.com/123,123.html，这时候后就需要条件模块来限制，同样的是使用正则找出请求的URL是不是有abc.com这个站点，不是得话，那就不转换。

###### 操作模块 ######
	
操作模块就是要把上面第一步正则匹配到的URL的信息加到你自己的信息下面。上图的{R:1},{R;2}这时候就可以加到你自己的URL里面了

pages/edu-detail.html?channelID={R:1}&newscode={R:2}

![](http://images-1253712676.costj.myqcloud.com/IIS%E9%87%8D%E5%86%99URL/IIS%E9%87%8D%E5%86%99RUL.2017060606.png)

点击保存，大功告成。

测试的时候，输入http://abc.com/news/1234,1231221.html会直接跳转到http://abc.com/pages/edu-detail.html?channelID=1234&newscode=1231221里面

> 这里只是初步研究URL重写，其他的功能待研究
	

 