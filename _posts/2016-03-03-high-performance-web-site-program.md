---
layout: postlayout
title: 《构建高性能web站点》笔记--应用程序篇
thumbimg: 63d9f2d3572c11dfe4b45162632762d0f703c20e.jpg
categories: [Web]
tags: [Performance, Web dev]
---

## 起因
大概花了一个月不到的时间，看完了这本400页不到的书《构建高性能web站点》，不得不说这是我第一次真正意义上完全看完一本书，尽管曾经看过许多技术类的书。其中一个原因，就是大部分的技术类书籍偏向枯燥，即使是本着某种虔诚的目的和愿望去阅读，仍然很容易中途放弃。但是这本书却不同，它十分能吸引我的阅读愿望，几乎在所有的环节上能够引起我的共鸣思考，于是便快速的阅读了一遍此书。作者主要以典型的LAMP为例子，我几乎没有接触过这方面，但是相信思想是一致的，学思想打基础才是关键。因此，本文是以概要性的总结为主。

## 概念

吞吐率：一个衡量web服务性能的指标，表征每秒处理请求的次数。该指标受到3方面的因素影响：并发用户数、总请求数、请求资源的类型。有时在请求总数一定的情况下，并发用户越多，吞吐率反而越高；另外，请求一个几kb的文件和请求一个几m的文件，最终完成处理的时间显然是不一样的。因此，吞吐率是一个比较综合的指标，并不是指并发能力。

## 概览图

下图简单描述了用户浏览器，代理服务器，web服务器和应用程序服务器的典型关系，当然通常应用程序服务器同时包含在web服务器中，这里为了区分职责而分开画了。

![](http://images.cnblogs.com/cnblogs_com/P_Chou/201210/201210092249354396.png)

## 缓存

构建高性能web站点时，抛开基础架构（数据库分区的问题也包括在基础架构中了），在应用程序、编码层面主要要考虑的问题就是缓存的设计，合理的缓存设计可以使提供动态网页服务的网站性能大幅度提高。当然，在架构阶段设计缓存解决方案，绝非简单的技术问题，需要从业务出发，再结合各种技术。下面按照一次HTTP请求的顺序，对每个环节的缓存设计从技术角度进行讨论。

### 客户端缓存

可以利用客户端浏览器的缓存机制，来减少浏览器对服务端的请求次数（当然在服务端进行图片等资源合并，并结合css图片定位技术，也可以减少HTTP请求），利用好HTTP的缓存协商，可以设计出灵活的客户端缓存方案。在HTTP头中下面的内容与缓存协商有关：

`Last-Modified`：动态页面通过主动推送该值，暗示浏览器在下次请求同一个url的时候，优先使用If-Modified-Since值与服务端进行缓存协商，如果缓存没有过期，那么服务端可以不用重新计算动态网页，通过返回304通知浏览器。网站的静态资源往往使用这种方法。但是该方法有一个缺点：有时，文件的最后更改时间虽然改了，但是内容却没有变，这样无法充分发挥浏览器缓存的能力。

`ETag`：Web服务器为每个url生成一个散列值，增加在HTTP头的ETag标记中，浏览器会优先使用If-None-Match加上这个散列值来协商缓存过期。通过对静态文件的内容进行md5变换，可以生成散列值，这样可以弥补Last-Modified的不足。但是随之带来的是服务端md5变换的计算开销。

`Expires`：上述两种方式，虽然可以使服务端多少避免了反复的动态网页解析和计算，但浏览器还是必须通过HTTP请求来进行协商，并没有真正意义上减少请求的次数。通过在HTTP头中添加Expires标记可以明确的告知浏览器过期形式，浏览器会彻底减少请求的次数。


### 反向代理缓存

在web服务器前端，还有反向代理服务器缓存。反向代理服务器本质上就是代理服务器，只是将外网的请求转发给内网的web服务器处理，他们都工作在应用层，能够理解HTTP协议。正向代理服务器具有HTTP缓存、HTTP过滤等功能，反向代理服务器同样具有HTTP缓存的能力，而且还具备一定程度上的安全性。一切HTTP友好的动态程序同样能够很好的在反向代理服务器上实现缓存。重量级的squid、轻量级的varnish、甚至是Nginx这样的web服务器软件，都可以胜任反向代理服务。

上述的代理服务器软件产品，通过各种配置可以缓存基于HTTP协议的web响应。


### Web服务器缓存

Web服务器有可能支持基于url的缓存（基于key-value对），这类似反向代理缓存。缓存通常可以通过配置存储在内存或磁盘上，在缓存有效期的问题上，通常是基于HTTP协议中的头部信息判断。但是使用这样的机制需要注意：

动态程序会可能变得依赖于特定的web服务器
注意编写面向HTTP缓存友好的动态程序，会使你的动态程序更有生命力
web 服务器还具有缓存文件描述符（类似句柄）的能力，这样可以减少文件的open操作，同样是一种减少系统调用的措施，这对于一些小文件有些效果，因为文件越小花在open上的开销将越来越占有重要的比例。  

### 应用程序缓存

应用程序本身可以对动态内容进行缓存，这可以体现在三个层面：

- 动态脚本缓存：每次脚本解析都需要消耗一定的时间，为了加快这个速度，一些服务器端脚本语言都支持动态脚本的预编译，比如我们熟悉的ASP.NET、JSP都有所谓的中间语言，解析这些中间语言会快很多。
- 动态脚本框架支持的缓存：一些动态脚本框架支持缓存，同样在选型的时候要关注一下这些框架的缓存机制和原理，比如缓存如何存放，过期如何设置，是否支持局部缓存等。
- 应用程序自身实现缓存：应用程序根据特定的业务需求独立设计缓存。

在缓存设计的具体技术上有以下几点：

- 缓存在内存，这种方式的优点就是减少了磁盘的读写，但是内存有限，单机不易扩展。
- 缓存在分布式缓存，这种方式的优点是减少了磁盘读写，并提高了可靠性和扩展性，但是由于需要网络IO，性能稍逊于单机缓存。
- 局部页面缓存，对于一些页面无法完整缓存的，可以考虑局部缓存。
- 静态化，将网站静态化可以极大的提高性能，因为用户的请求不需要动态脚本处理。

### 服务器系统能力的制约因素

这部分内容对于所有的服务器（无论是代理服务器、web服务器还是其他），都具有普遍适用的意义。

- 多进程、多线程的选择和调度：进程切换和线程切换都需要一定的系统开销，通常使用多线程模型的web服务器软件比使用多进程，具备更优的性能。
- 系统调用：一些需要从用户模式切换到内核模式的函数调用可以称为系统调用，比如：打开文件。系统调用会有一定程度上的开销，减少系统调用是可以加快处理速度的程序设计细节。
- TCP链接保持：可以通过保持TCP链接来减少服务端和客户端之间的创建和关闭TCP链接的操作。HTTP中的Connection:Keep-Alive就有这样的功能
- IO模型：由于CPU的速度远远比IO快，IO延迟往往成为性能瓶颈，因此，IO模型十分重要。

各种IO模型：

> PIO：CPU直接干预磁盘和内存的数据交互，即无论是数据从内存到磁盘还是磁盘到内存都要经过CPU寄存器。这样的模型，可想而知，CPU有很多时间都需要等待慢速设备。
>
> DMA（Direct Memory Access）：CPU通过向DMA控制器发送指令来控制处理数据，数据处理完之后通知CPU，这可以很大程度上释放CPU资源。
>
> 同步阻塞I/O：对于进程来说，一些系统调用为了同步IO，会不同程度上阻塞进程，比如accept、send、read等。
>
> 同步非阻塞I/O：对于进程来说，一些系统调用可以在调用完之后立即返回，告知进程IO是否就绪，避免阻塞进程。
>
> 多路I/O就绪通知：对于同步非阻塞I/O的方式，进程仍然需要轮询文件描述符（句柄）来得知哪些IO就绪了，而多路I/O就绪通知将这个过程改成回调通知。
>
> 内存映射：将文件与内存的某块地址空间相映射，这样可以想写内存一样写文件。当然这种方式本质上跟写文件没有什么区别。
>
> 直接I/O：在用户进程地址空间和磁盘中间通常都会有操作系统管辖的内核缓冲区，当写入文件时，一般是写入这个缓冲区，然后由一些延迟策略来写入磁盘。这样做可以提高写效率。但是对于诸如数据库这样的应用来说，往往希望自己管理读写缓存，避免内核缓冲区的无畏内存浪费。Linux的open函数支持O_DIRECT参数来进行直接IO。
>
> sendfile：如果web服务器想发送一个文件，将会经历如下过程：打开文件，从磁盘中读取文件内容（这通常涉及到内核缓冲区数据复制到用户进程），然后进程通过socket发送文件内容（这通常设计到用户进程数据复制到网卡内核缓冲区），可以看到重复的数据复制是可以避免的。sendfile可以支持直接从文件内核缓冲区复制到网卡内核缓冲区。

## 总结

本文从应用程序层面总结了构建高性能web站点的要点，主要探讨了缓存的实现技术。

劳动果实，转载请注明出处：[《构建高性能web站点》笔记--应用程序篇]({% post_url 2016-03-03-high-performance-web-site-program %})

相关文章

> [《构建高性能web站点》笔记--基础架构篇]({% post_url 2016-03-04-high-performance-web-site-infrastructure %})
>
> 《构建高性能web站点》笔记--应用程序篇