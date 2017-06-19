title: WebApi跨域
date:  2017-05-12
tags: iis
categories: 
   - 后端
   - iis   
------

在接口调用的时候，需要考虑到接口的跨域请求。

在web.config配置中添加几条配置即可

``` bash
<system.webServer>        
	<httpProtocol>
		<customHeaders>
		    <add name="Access-Control-Allow-Origin" value="*" />
		    <add name="Access-Control-Allow-Methods" value="POST,OPTIONS" />
		    <add name="Access-Control-Allow-Headers" value="content-type" />
		</customHeaders>
	</httpProtocol>
</system.webServer>
```

在部署到IIS上时，可以在HTTP响应标头里查看配置是否存在，如果不存在的话，可以手动添加

用js来调用：

``` bash
<html>
    <head>
        <meta charset="utf-8">
        <script src="jquery-3.1.1.min.js"></script>
    </head>
    <body>
        <button>向页面发送 HTTP POST 请求，并获得返回的结果</button>
        <script>
        $.support.cors = true;
        $(document).ready(function(){
        $("button").click(function(){
            var url='接口地址';
            var sendInfo='接口请求信息';
            $.ajax({
            url:url,
            type:'POST',
            dataType:'json',
            //contentType:'application/json',     //添加这句会出错 
            data:sendInfo,
            success:function (data)
            {
                console.log(data);
            }
            });
            // $.post(url,JSON.stringify(sendInfo), 
            //   function (data){
            //     console.log(JSON.stringify(data))
            //   },"json"
            // );
            })
                
        });
        </script>
    </body>
</html>
```

> 调试发现可以跨区请求