---
layout: post
title: 高并发
tags:  高并发
categories: 读书笔记
---

#网站高并发处理


**防盗链怎么做？**

> 非技术手段 ： 给图片添加水印logo
> 技术手段   ： 利用重写规则。判断请求头里面的referer是否含有本网站域名，如果没有则禁止访问或者重写成其他地址。但是还是可以用curl伪装curl。


**怎么应对网站高并发？？？**
> 主要处理方式就是分流。
> 从架构上可以采用分层架构，增加服务器集群，并使用负载均衡来减轻web服务器的压力，数据库服务器采用读写分离，主从复制的方式，来减轻数据库服务器的压力。
> 从技术上可以优化的手段比较多：
> - 尽量减少http请求：合并图片，像外部js文件可以合并在一起、资源分布部署、防盗链等
> - 服务器端采用缓存技术（速度 ：内存缓存>静态缓存>查询数据库）
> - 合理设计数据库，使用高效sql语句，合理建立索引等。

大流量（访问网站的量）解决方案？
> - 减少http请求（比如合并图片）
> - 配置缓存，让浏览器缓存更新不频繁资源--现在浏览器一般默认是缓存的，不用单独配置。
> - 配置压缩，服务器端压缩，客户端解压
> - 把比较占用流量的资源单独部署
> - 防盗链
> - 买带宽

大存储的解决方式？
> - 缓存技术（页面静态化+内存缓存技术）
> - 数据库方面的优化




怎么配置MPM？？？
> 在httpd.conf里开启，MPM扩展  : Include conf/extra/httpd-mpm.conf在mpm的辅助配置文件（extra/httpd-mpm-conf）中进行配置大部分网站，中型网站推荐配置如下 ：


{% highlight php startinline linenos %} 
	<IfModule mpm_prefork_module>
		StartServers?5  		   #预先启动
		MinSpareServers?5
		MaxSpareServers?10 		#最大空闲进程
		ServerLimit?1500  		 #用于修改apache编程参数
		MaxClients?1000  		  #最大并发数
		MaxRequestsPerChild?0      #一个进程对应的线程数，对worker更效果。如果是0， 则不让进程死掉。 
	</IfModule>
{% endhighlight %}

> pv值如果百万级别：ServerLimit  2500		MaxClients 2000


怎么配置压缩，以减少传输的数据量？？？
> - 要开启gzip压缩的mod_deflate模块： LoadModule deflate_module modules/mod_deflate.so
> - 在虚拟主机里面针对具体的文件类型进行配置。例如：

{% highlight php startinline linenos %} 
	<ifmodule mod_deflate.c> 
	DeflateCompressionLevel 6       #压缩级别为6，可1-9，推荐为6 
	AddOutputFilterByType DEFLATE text/plain  #压缩文本文件 
	AddOutputFilterByType DEFLATE text/html  #压缩html文件 
	AddOutputFilterByType DEFLATE text/xml   #压缩xml文件
	</ifmodule>
{% endhighlight %}
	
> 压缩是要耗费cup资源的，图片、视频等文件压缩效果不好，一般压缩文本型文件即可。

