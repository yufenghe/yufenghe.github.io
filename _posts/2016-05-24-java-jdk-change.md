---
layout: postlayout
title: 解决安装多个jdk版本，修改本地变量无法生效的问题
description: 开发中机器上需要安装多个jdk版本，但是修改环境变量无法生效，java -version依然显示原先版本
thumbimg: 1346208288725.jpg
categories: [java]
tags: [java, jdk]
---

## jdk变更

本机新安装了jdk1.7，之前项目依赖jdk1.6，于是本机同时存在jdk1.6和jdk1.7两个版本。 
修改环境变量JAVA_HOME指向新的版本路径，但是java -version得到的结果依然显示为1.6的版本

看上去，新的环境变量JAVA_HOME=D:\Java\jdk1.7.0_79并没有生效。 
在安装JDK1.6时，自动将java.exe、javaw.exe、javaws.exe三个可执行文件复制到了C:\Windows\System32目录，由于这个目录在WINDOWS环境变量中的优先级高于JAVA_HOME设置的环境变量优先级。
解决方案：将java.exe,javaw.exe,javaws.exe替换为新版本jdk的对应文件，再执行java -version时，就得到了期望中的结果

```shell
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

在多版本同时存在的情况下，删除了C:\Windows\System32和C:\Windows\SysWOW64两个目录中的上述三个文件仍不能正常使用eclipse，可以使用以下这种方式：
在eclipse.ini文件中添加

```
-vm
D:\Java\jdk1.7.0_79_x86\bin\javaw.exe
```

可以添加copy命令将java.exe、javaw.exe、javaws.exe三个文件复制到C:\Windows\System32目录下：

```shell
xcopy D:\Java\jdk1.7.0_79\bin\java.exe C:\Windows\System32 /R /Y
xcopy D:\Java\jdk1.7.0_79\bin\javaw.exe C:\Windows\System32 /R /Y
xcopy D:\Java\jdk1.7.0_79\bin\javaws.exe C:\Windows\System32 /R /Y
```

参考资料：
>[1] http://blog.csdn.net/cuidiwhere/article/details/12362829
>
>[2] http://www.oschina.net/code/snippet_145965_43181
