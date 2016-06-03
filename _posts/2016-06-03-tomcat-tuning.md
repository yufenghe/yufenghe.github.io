---
layout: postlayout
title: tomcat配置优化
description: 设置tomcat内存使用配置，针对tomcat优化的一些参数设置
thumbimg: 1346208288725.jpg
categories: [tomcat]
tags: [java, tomcat]
---
参考：

- [`http://mp.weixin.qq.com/s?__biz=MzA3MzYwNjQ3NA==&mid=2651296654&idx=1&sn=b04fc6cecfcca64f9e48220af5e1e6c1&scene=4#wechat_redirect`](http://mp.weixin.qq.com/s?__biz=MzA3MzYwNjQ3NA==&mid=2651296654&idx=1&sn=b04fc6cecfcca64f9e48220af5e1e6c1&scene=4#wechat_redirect)
- [`http://blog.163.com/drg_king/blog/static/1761515612012864140199/`](http://blog.163.com/drg_king/blog/static/1761515612012864140199/)

以下参数配置都是针对windows平台的参数配置，linux环境下配置大体相同。此处tomcat配置环境为`apache-tomcat-7.0.47`

## jvm内存设置

Tomcat内存优化主要是对启动参数优化，我们可以在tomcat的启动脚本`catalina.bat`中设置`JAVA_OPTS`参数。

```xml
set JAVA_OPTS=-Djava.awt.headless=true -server -Xms1024m -Xmx1024m -XX:NewSize=256m -XX:MaxNewSize=256m -XX:PermSize=128m -XX:MaxPermSize=128m -XX:+DisableExplicitGC
```
参数说明：

```xml
-server  				/*启用jdk 的 server 版*/
-Xms      				/*java虚拟机初始化时的最小内存*/
-Xmx      				/*java虚拟机可使用的最大内存*/
-XX:PermSize    		/*Perm区初始化的内存大小*/
-XX:MaxPermSize   		/*Perm（俗称方法区）占整个堆内存的最大值*/
-XX:NewSize				/*新生代预估上限的默认值*/
-XX:MaxNewSize			/*新生代占整个堆内存的最大值*/
-XX:+DisableExplicitGC	/*禁用System.gc()*/
```
在使用NIO框架（Netty、Mina）时，会使用Direct Buffer，可能会造成内存无法回收，而System.gc()被外部禁用，就会导致对象无法及时回收，相应的native memory不能被释放，可将`-XX:+DisableExplicitGC`去掉，换上`-XX:+ExplicitGCInvokesConcurrent`，使得外部显示调用System.gc()有效。

设置Tomcat启动的初始内存，其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)是物理内存的1/4。可以利用JVM提供的-Xmn -Xms -Xmx等选项，要加“m”说明是MB，否则就是KB了，在启动tomcat时会报内存不足。

提示：Heap Size 最大不要超过可用物理内存的80％，一般的要将-Xms和-Xmx选项设置为相同，PermSize和MaxPermSize选项设置相同，避免再次重新分配，而-Xmn为1/4的-Xmx值。 这两个值的大小一般根据需要进行设置。

## tomcat并发优化

默认的tomcat 参数：

```xml
<Connector port=“8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```

修改为：

```xml
<Connector port=“8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
  maxThreads="600"
  minSpareThreads="25"
  maxSpareThreads="100"
  acceptCount="700"
  connectionTimeout="20000"
  compression="on"
  compressionMinSize="2048"   
  noCompressionUserAgents="gozilla, traviata"   
  compressableMimeType="text/html,application/xml,application/json,application/javascript,text/css,text/plain"  
  disableUploadTimeout="false"
  connectionUploadTimeout="60000"
  URIEncoding="UTF-8"
  redirectPort="8443" />
```

- protocol="org.apache.coyote.http11.Http11NioProtocol"  //使用java的异步io护理技术,no blocking IO
- maxThreads=“600" 表示最多同时处理600个连接 			   //最大线程数
- minSpareThreads=“100" 								   //表示即使没有人使用也开这么多空线程等待  ///初始化时创建的线程数
- maxSpareThreads=“500"                                  //表示如果最多可以空500个线程，一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。
- acceptCount="700"									   //当线程数达到maxThreads后，后续请求会被放入一个等待队列，这个acceptCount是这个队列的大小，如果这个队列也满了，就直接refuse connection
- minProcessors="5"									   //最小空闲连接线程数，用于提高系统处理性能，默认值为10
- maxProcessors="1000"								   //最大连接线程数，即：并发处理的最大请求数，默认值为75
- enableLookups="false"								   //是否反查域名，为了消除DNS查询对性能的影响我们可以关闭DNS查询，取值为：true或false。为了提高处理能力，应设置为false
- compression="on" 									   //打开压缩功能，HTTP 压缩可以大大提高浏览网站的速度，它的原理是，在客户端请求网页后，从服务器端将网页文件压缩，再下载到客户端，由客户端的浏览器负责解压缩并浏览。相对于普通的浏览过程HTML,CSS,Javascript , Text ，它可以节省40%左右的流量。更为重要的是，它可以对动态生成的，包括CGI、PHP , JSP , ASP , Servlet,SHTML等输出的网页也能进行压缩，压缩效率惊人。
- compressionMinSize="2048" 							   //启用压缩的输出内容大小，这里面默认为2KB
- noCompressionUserAgents="gozilla, traviata" 		   //对于以下的浏览器，不启用压缩
- compressableMimeType="text/html,text/xml"　          //压缩类型

其中minProcessors、maxProcessors与minSpareThreads,maxThreads意义差不多。
最后不要忘了把8443端口的地方也加上同样的配置，因为如果我们走https协议的话，我们将会用到8443端口这个段的配置

另：也可以使用连接池方式

```xml
<Executor name="tomcatThreadPool" 
        namePrefix="tomcatThreadPool-" 
        maxThreads="1000" 
        maxIdleTime="300000"
        minSpareThreads="200"/>
<Connector executor="tomcatThreadPool"
           port="20003" protocol="HTTP/1.1"
           acceptCount="800"
           minProcessors="300"
           maxProcessors = "1000"
           redirectPort="8443" />
```
## 查看tomcat连接数

假设服务器上开启了 2个tomcat实例,分别监听8040和8050端口

```shell
netstat -na | grep ESTAB | grep 8040 | wc -l
netstat -na | grep ESTAB | grep 8050 | wc -l
```
二者之和,就是所有tomcat的连接数 

## 查看tomcat内存

在conf文件夹下的tomcat-users.xml文件中添加用户，比如

```xml
<role rolename="tomcat"/>
<role rolename="role1"/>
<role rolename="manager"/>
<role rolename="admin"/>
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/> 
<user username="tomcat" password="tomcat" roles="admin,manager,role1,tomcat,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
```
访问http://localhost:port/manager/status，输入上面添加的用户名和密码,效果图如下：
![1]({{ site.BASE_PATH }}/assets/img/20160603162824-tomcat-memory.png)

## 附：jvm其他参数

-Xss
是指设定每个线程的堆栈大小。这个就要依据你的程序，看一个线程 大约需要占用多少内存，可能会有多少线程同时运行等。一般不易设置超过1M，要不然容易出现out ofmemory。

-XX:+AggressiveOpts
作用如其名（aggressive），启用这个参数，则每当JDK版本升级时，你的JVM都会使用最新加入的优化技术（如果有的话）

-XX:+UseBiasedLocking
启用一个优化了的线程锁，我们知道在我们的appserver，每个http请求就是一个线程，有的请求短有的请求长，就会有请求排队的现象，甚至还会出现线程阻塞，这个优化了的线程锁使得你的appserver内对线程处理自动进行最优调配。

-XX:+UseParNewGC
对年轻代采用多线程并行回收，这样收得快。

-XX:+UseConcMarkSweepGC
即CMS gc，这一特性只有jdk1.5即后续版本才具有的功能，它使用的是gc估算触发和heap占用触发。
我们知道频频繁的GC会造面JVM的大起大落从而影响到系统的效率，因此使用了CMS GC后可以在GC次数增多的情况下，每次GC的响应时间却很短，比如说使用了CMS GC后经过jprofiler的观察，GC被触发次数非常多，而每次GC耗时仅为几毫秒。

-XX:MaxTenuringThreshold
设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概率。这个值的设置是根据本地的jprofiler监控后得到的一个理想的值，不能一概而论原搬照抄。

-XX:+CMSParallelRemarkEnabled
在使用UseParNewGC 的情况下, 尽量减少 mark 的时间

-XX:+UseCMSCompactAtFullCollection
在使用concurrent gc 的情况下, 防止 memoryfragmention, 对live object 进行整理, 使 memory 碎片减少。

-XX:LargePageSizeInBytes
指定 Java heap的分页页面大小

-XX:+UseFastAccessorMethods
get,set 方法转成本地代码

-XX:+UseCMSInitiatingOccupancyOnly
指示只有在 oldgeneration 在使用了初始化的比例后concurrent collector 启动收集

-XX:CMSInitiatingOccupancyFraction=70
CMSInitiatingOccupancyFraction，这个参数设置有很大技巧，基本上满足(Xmx-Xmn)*(100- CMSInitiatingOccupancyFraction)/100>=Xmn就不会出现promotion failed。在我的应用中Xmx是6000，Xmn是512，那么Xmx-Xmn是5488兆，也就是年老代有5488 兆，CMSInitiatingOccupancyFraction=90说明年老代到90%满的时候开始执行对年老代的并发垃圾回收（CMS），这时还 剩10%的空间是5488*10%=548兆，所以即使Xmn（也就是年轻代共512兆）里所有对象都搬到年老代里，548兆的空间也足够了，所以只要满 足上面的公式，就不会出现垃圾回收时的promotion failed；因此这个参数的设置必须与Xmn关联在一起。

-Djava.awt.headless=true
这个参数一般我们都是放在最后使用的，这全参数的作用是这样的，有时我们会在我们的J2EE工程中使用一些图表工具如：jfreechart，用于在web网页输出GIF/JPG等流，在winodws环境下，一般我们的app server在输出图形时不会碰到什么问题，但是在linux/unix环境下经常会碰到一个exception导致你在winodws开发环境下图片显示的好好可是在linux/unix下却显示不出来，因此加上这个参数以免避这样的情况出现。