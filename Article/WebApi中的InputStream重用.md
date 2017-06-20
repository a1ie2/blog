title: WebApi中的InputStream重用
date:  2017-06-20
tags: c#
categories: 
   - 后端
   - c#
   - webapi   
------

在写接口的时候因为有文件上传的功能，所以就用到了`HttpContext.Current.Request.Files["File"].InputStream`和FTP来实现上传，但后来又要加上计算文件的MD5值。后来就发现了这个问题。

因为我的FTP是要读取流的，但是读取完之后，就把流关掉了，导致我拿流计算MD5的时候计算的永远是空流的MD5，然后才发现，每次上传的文件MD5都是一样的。

后来换了个顺序，先计算MD5，然后再上传文件，但是这样也有另一个问题，因为计算MD5结束，虽然流没有关掉，但是因为读取了流，导致指针是在流的最后一位，这样上传上去的文件永远是0kb。


后来在调试的时候，发现了上面的问题，开始着手解决

- 思路一

把`HttpContext.Current.Request.Files["File"].InputStream`复制两份，一份上传，一份计算MD5，代码类似如下

``` bash
  Stream stream4Upload = HttpContext.Current.Request.Files["File"].InputStream;
  Stream stream4MD5 = HttpContext.Current.Request.Files["File"].InputStream;
```

结论：

因为Stream没有实现接口`ICloneable`，导致就算代码这么写，也没有用，两个流指向的都是同一个流

- 思路二

因为我在上传FTP之后会返回文件在服务器上面的地址，所以，这时候可以拿这个地址再去读取文件流计算出MD5

结论：

可行，但是上传之后再读取，这样不大合适

- 思路三

因为stream没有实现接口`ICloneable`，但是如果我可以new出两个stream,这样也可以实现我的逻辑，所以我不直接使用`HttpContext.Current.Request.Files["xxx"].InputStream`，而是在使用之前读出来，然后分别赋值到两个Stream中。

``` bash
 var file = HttpContext.Current.Request.Files["File"];
 byte[] buffer = new byte[file.ContentLength];
 Stream stream = file.InputStream;
 stream.Read(buffer, 0, (int)file.ContentLength);
 Stream stream4Upload = new MemoryStream(buffer);
 Stream stream4MD5 = new MemoryStream(buffer);
 FTPHelper ftp = new FTPHelper();
 ftp.Upload(stream4Upload, "", "123");
 string MD5Str= MethodHelper.GetMD5(stream4MD5);
```

结论：

可行，但是这样其实和思路二没啥区别

- 思路四

因为我在计算MD5的时候只是读取流，并没有关闭流。所以在读取完之后，只需要把指针指向到开始位就行了

``` bash
  Stream stream = file.InputStream;
  string MD5Str = MethodHelper.GetMD5(stream).ToUpper();
   stream.Seek(0, SeekOrigin.Begin);
```

加上一行`stream.Seek(0, SeekOrigin.Begin)`

结论:

在实际测试的时候是完全可以的，这种情况下只适用于你的流没有close掉，如果close掉之后，把指针指向开始位也不行，因为里面的都是0


至此，踩坑结束