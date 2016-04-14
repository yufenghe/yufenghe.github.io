---
layout: postlayout
title: Github使用jekyll建站过程踩过的坑
description: 使用jekyll模板引擎创建Github博客中遇到的问题
thumbimg: 1346208288725.jpg
categories: [web-build]
tags: [github-page, jekyll, liquid]
---


创建博客具体过程不再细表，可参见[一步步在GitHub上创建博客主页(1)]({% post_url  2016-03-10-build-github-blog-page-01 %})。

因为博客使用jekyll模板引擎，需要安装Ruby，选择安装的Ruby版本为`ruby 2.2.4p230 (2015-12-16 revision 53155) [x64-mingw32]`，RubyDevKit版本按照Ruby的版本要求安装，gem版本为`2.4.5.1`，在这个过程中遇到过好多问题，要非常感谢`Stack Overflow`和`Ruby China`，总有一款答案适合自己。

## gem来源

安装gem后默认的数据源为[`https://rubygems.org/`](https://rubygems.org/)，因为放在Amazon s3的原因，国内无法访问，具体原因大家懂得，所以就设置了[`https://ruby.taobao.org/`](https://ruby.taobao.org/)，谁知在执行`gem install gemfile`的时候报错：

```ruby
ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
    SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certif
icate verify failed (https://rubygems-china.oss-cn-hangzhou.aliyuncs.com/gems/ff
i-1.9.10-x64-mingw32.gem)
```
在[`Ruby China`](https://ruby-china.org/topics/29323)上找到了答案，将数据源换成[`https://gems.ruby-china.org/`](https://gems.ruby-china.org/)，原来是因为`ruby 没有包含 SSL 证书，所以 https 的链接被服务器拒绝`，，解决方法很简单，首先在这里下载证书 [`http://curl.haxx.se/ca/cacert.pem`](http://curl.haxx.se/ca/cacert.pem), 然后再环境变量里设置 `SSL_CERT_FILE` 这个环境变量，并指向 `cacert.pem `文件，问题圆满解决，终于结束了这该屎的SSL报错问题：

```sh
来源：https://gist.github.com/fnichol/867550
The Ruby Way! (Fun)
This assumes your have already installed the Rails Installer for Windows.
Download the ruby script to your Desktop folder from https://gist.github.com/raw/867550/win_fetch_cacerts.rb. Then in your command prompt, execute the ruby script:
ruby "%USERPROFILE%\Desktop\win_fetch_cacerts.rb"

Now make ruby aware of your certificate authority bundle by setting SSL_CERT_FILE. To set this in your current command prompt session, type:
set SSL_CERT_FILE=C:\RailsInstaller\cacert.pem

To make this a permanent setting, add this in your control panel.

The Manual Way (Boring)
Download the cacert.pem file from http://curl.haxx.se/ca/cacert.pem. Save this file to C:\RailsInstaller\cacert.pem.
Now make ruby aware of your certificate authority bundle by setting SSL_CERT_FILE. To set this in your current command prompt session, type:
set SSL_CERT_FILE=C:\RailsInstaller\cacert.pem

To make this a permanent setting, add this in your control panel.

```

```sh
$ gem sources -a https://gems.ruby-china.org/ --remove https://ruby.taobao.org/
```

Or install it yourself as:

```sh
$ gem install html-pipeline
```
因为项目中使用了分页，并且jekyll版本为3.0.3，所以必须依赖jekyll-paginate组件
```sh
在_config.yml 中添加 gems: [jekyll-paginate]
```
可以使用`jekyll build --incremental --profile --trace`查看整个构建过程

由于jekyll为3.0.3的关系，高亮插件pygments会报错，建议使用`rouge`
```sh
$ gem install rouge
修改_config.yml文件（highlighter: rogue）
```