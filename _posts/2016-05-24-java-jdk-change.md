---
layout: postlayout
title: 解决安装多个jdk版本，修改本地变量无法生效的问题
description: 开发中机器上需要安装多个jdk版本，但是修改环境变量无法生效，java -version依然显示原先版本
thumbimg: 1346208288725.jpg
categories: [java]
tags: [java, jdk]
---

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

网上找到一个脚本设置环境变量的方式：

```shell
@echo off 
 
:init 
set JAVA_HOME_1_7=C:\Program Files\Java\jdk1.7.0_22 
set JAVA_HOME_1_6=D:\Program Files\Java\jdk1.6.0_27 
set Eclipse_EXE=D:\Tools\DEV(eclipse3.3)\eclipse.exe 
 
:start 
echo JDK 版本: 
java -version 
ping 127.0.0.1 -n 2 -w 1000 > nul 
echo. 
echo ============================================= 
echo jdk版本列表 
echo 1.7 
echo 1.6 
echo ============================================= 
 
:select
set /p opt=请选择jdk版本： 
if %opt%==1.7 ( 
  start  /I /WAIT /B wmic ENVIRONMENT where name='JAVA_HOME' set VariableValue="%JAVA_HOME_1_7%" >nul 
rem reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v JAVA_HOME /t reg_sz /d "%JAVA_HOME_1_7%" /f
  goto success 
) 
if %opt%==1.6 ( 
    start /I /WAIT /B wmic ENVIRONMENT where name='JAVA_HOME' set VariableValue="%JAVA_HOME_1_6%" >nul 
rem reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v JAVA_HOME /t reg_sz /d "%JAVA_HOME_1_6%" /f
    goto success 
) 
echo 选择的版本错误,请重新选择！ 
PAUSE 
goto start 
 
:success 
echo. 
echo 设置环境变了成功. 
ping 127.0.0.1 -n 2 -w 1000 > nul 
taskkill /f /im eclipse.exe 
ping 127.0.0.1 -n 3 -w 1000 > nul 
start %Eclipse_EXE%
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
