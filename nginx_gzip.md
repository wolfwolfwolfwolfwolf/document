#nginx开启压缩

versin: 1.0
author: wolf
data: 20181011



##一、指令放置的位置

###1.http,server,location


##二、指令模板
          gzip on; 
          gzip_buffers 4 16k;
          gzip_comp_level 6;
          gzip_http_version 1.1;
          gzip_min_length 1k;
          gzip_types ext/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
          gzip_disable "MSIE [1-6].";
          gzip_vary on;


##三、指令解释

###1.gzip on  
开启压缩

###2. gzip_buffers 4 16k;

       设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 
       4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。

###3. gzip_comp_level 6;
       # gzip压缩比/压缩级别，压缩级别 1-9，级别越高压缩率越大，当然压缩时间也就越长（传输快但比较消耗cpu）。


###4. gzip_http_version 1.1;
    默认值: gzip_http_version 1.1(就是说对HTTP/1.1协议的请求才会进行gzip压缩)
    识别http的协议版本。由于早期的一些浏览器或者http客户端，可能不支持gzip自解压，用户就会看到乱码，所以做一些判断还是有必要的。 
    注：99.99%的浏览器基本上都支持gzip解压了，所以可以不用设这个值,保持系统默认即可。
    假设我们使用的是默认值1.1，如果我们使用了proxy_pass进行反向代理，那么nginx和后端的upstream server之间是用HTTP/1.0协议通信的，如果我们使用nginx通过反向代理做Cache Server，而且前端的nginx没有开启gzip，同时，我们后端的nginx上没有设置gzip_http_version为1.0，那么Cache的url将不会进行gzip压缩

###5.gzip_min_length
         默认值:0 ，不管页面多大都压缩
         设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
         建议设置成大于1k的字节数，小于1k可能会越压越大。 即: gzip_min_length 1024


###6.gzip_types
      默认值: gzip_types text/html (默认不对js/css文件进行压缩)
      压缩类型，匹配MIME类型进行压缩
      不能用通配符 text/*
     (无论是否指定)text/html默认已经压缩 
      设置哪压缩种文本文件可参考 conf/mime.types


###7.gzip_disable "MSIE [1-6]."

    禁用IE6的gzip压缩，又是因为杯具的IE6。当然，IE6目前依然广泛的存在，所以这里你也可以设置为“MSIE [1-5].”
    IE6的某些版本对gzip的压缩支持很不好，会造成页面的假死，今天产品的同学就测试出了这个问题
    后来调试后，发现是对img进行gzip后造成IE6的假死，把对img的gzip压缩去掉后就正常了
    为了确保其它的IE6版本不出问题，所以建议加上gzip_disable的设置


###8.gzip_vary 

     和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持的也压缩，
     所以根据客户端的HTTP头来判断，是否需要压缩




##四、总结

###1.nginx服务器无法压缩代理的资源，只能压缩本地资源。
     列如：图片在OSS上，nginx开启压缩图片，但无法压缩这些图片。

###2.判断这个文件或者图片是否压缩
      a.	
      是否返回 Content-Encoding: gzip
      b.查看网页 Response Headers 是否有 content-encoding: gzip


