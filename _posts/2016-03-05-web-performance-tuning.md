---
layout: postlayout
title: web应用程序调优
thumbimg: BMCAtriumCMDB.png
categories: [Web]
tags: [Performance]
---


web应用程序调优要分两个层面：

- 调优web架构和配置。例如JVM参数，servlet engine线程配置，HTTP协议参数等这些不在web应用程序的可控范围内的调优。
- 调优mid-tier应用程序本身。


## Web基础配置调优 ##

下面的参数将有效的改善浏览器的响应时间：

### HTTP keep-alive ###

允许HTTP协议中的保持连接的数量，这将减少客户端与服务端建立TCP连接的次数。客户端将能够用现有的连接来进行HTTP通信。通常的web server在这个参数上需要考虑两个方面：

1. Keep-alive count：表示服务器能够保持连接的最大数量，对于Apache，默认100
2. Connection timeout：表示对于空闲的TCP连接，多长时间超时并关闭，对于Apache，默认20sec

这两个参数都会通过HTTP响应头返回浏览器，浏览器会进行合适的处理。默认的参数对于像mid-tier这样的应用并不适用，因为mid-tier应用是基于AJAX类型的应用，大量的AJAX请求却只包含少量的数据，如果反复的建立连接，可想而知。推荐设置如下

{% highlight xml %}

<Connector URIEncoding="UTF-8" acceptCount="100" 
connectionTimeout="90000"  
maxHttpHeaderSize="8192" maxKeepAliveRequests="-
1" maxThreads="500" port="80" 
protocol="HTTP/1.1" redirectPort="8443"/>

{% endhighlight %}

以上设置对性能提升的幅度取决于很多因素，其中一点就是网络延时，网络延时越大，这种设置的效果越好（大大减少TCP握手）。并且，对于HTTPS的站点此种提升可能更大（减少了SSL建立时期的密钥协商）。利用web代理调试技术可以观察设置是否生效，例如Fiddler。

 

### JVM settings

包括`JVM heap size`、`MaxPermSize`。这些值将影响垃圾回收的行为。不够优化的设置会导致以下问题：

- 垃圾回收更频繁，这样消耗更多的CPU
- 内存不能被完整的分配

以下是推荐配置：

![]({{ site.BASE_PATH }}/assets/img/bmc-performance-tuning-img0.png)

上图的配置仅仅考虑了你的Tomcat只运行midt-tier应用程序。如果还有其他应用共享一个Tomcat，你可能需要增加JVM堆的最大值和最小值

如果你有足够的内存空间，可以通过配置`<MT>/WEB-INF/classes/config.properties`中的`arsystem.ehcache.referenceMaxElementsInMemory`配置项。这些项目决定了mid-tier缓存AR系统的表单等以提高性能。每增加700M的内存，可以对该值增加1250。（For each additional 700 MB, you can increase the arsystem.ehcache.referenceMaxElementsInMemory by 1250 MB）其余类似该项目的配置表示对不同类型的对象缓存的权重，比如默认值下，缓存的active link数量最大可以是1250×4.904=6130个。

对于64的JVM参见：<http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#64bit_performance>

BMC内部的性能压力测试表明，32位JVM的性能在CPU使用上优于64位JVM至少45％。如果你使用64位的JVM，考虑Oracle的建议，使用混合模式和并行垃圾回收。

- XX:+UseCompressedOops
- -XX:+UseParallelGC
 

### Web应用程序的宿主线程配置

考虑下面的参数，具体配置可以参考上文：

- maxThreads：Tomcat最大同时处理HTTP请求的数量
- acceptCount：如果Tomcat无法第一时间处理请求，请求将进入队列。这个参数表征请求队列的最大容量
