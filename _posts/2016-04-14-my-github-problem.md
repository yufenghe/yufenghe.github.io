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

## gem源无法下载包问题

安装gem后默认的数据源为[`https://rubygems.org/`](https://rubygems.org/)，因为放在Amazon s3的原因，国内无法访问，具体原因大家懂得，所以就设置了[`https://ruby.taobao.org/`](https://ruby.taobao.org/)，谁知在执行`gem install gemfile`的时候报错：

```ruby
    ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
        SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certif
    icate verify failed (https://rubygems-china.oss-cn-hangzhou.aliyuncs.com/gems/ff
    i-1.9.10-x64-mingw32.gem)
```
在[`Ruby China`](https://ruby-china.org/topics/29323)上找到了答案，将数据源换成[`https://gems.ruby-china.org/`](https://gems.ruby-china.org/)，原来是因为`ruby 没有包含 SSL 证书，所以 https 的链接被服务器拒绝`，，解决方法很简单，首先在这里下载证书 [`http://curl.haxx.se/ca/cacert.pem`](http://curl.haxx.se/ca/cacert.pem), 然后再环境变量里设置 `SSL_CERT_FILE` 这个环境变量，并指向 `cacert.pem `文件，问题圆满解决，终于结束了这该屎的SSL报错问题：

```ruby
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

在命令行中执行下列命令：
   
```bash
     $ gem sources -a https://gems.ruby-china.org/ --remove https://ruby.taobao.org/
```

## 引用分页无法启动的问题

报错：

```ruby
    Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration file.
```
因为项目中使用了分页，并且jekyll版本为3.0.3，需要在_config.yml中添加对jekyll-paginate的依赖

```ruby
    gems: [jekyll-paginate]
```
可以使用以下命令查看整个构建过程：

```ruby
    jekyll build --incremental --profile --trace
```

## 高亮插件问题

Github升级Jekyll到3.0… GitHub Pages now faster and simpler with Jekyll 3.0. 修改后发布到Github从提交修改到生成页面, 比以前要快了. 这是好事.。
这次升级最大的两个变化是：
>强制只能使用kramdown做markdown解释。
>强制只能使用rouge做语法解释(原来是pygments)。

另外还不再支持relative_permalinks和Textile了.
之前使用的高亮插件为`pygments`，虽然构建成功，但是Github会发送警告。
报警邮件如下：

```ruby
The page build completed successfully, but returned the following warning:
 
You are attempting to use the 'pygments' highlighter, which is currently unsupported on GitHub Pages. Your site will use 'rouge' for highlighting instead. To suppress this warning, change the 'highlighter' value to 'rouge' in your '_config.yml'. For more information, see https://help.github.com/articles/page-build-failed-config-file-error/#fixing-highlighting-errors.
 
GitHub Pages was recently upgraded to Jekyll 3.0. It may help to confirm you're using the correct dependencies:
 
https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0
 
For information on troubleshooting Jekyll see:
 
https://help.github.com/articles/troubleshooting-jekyll-builds
 
If you have any questions you can contact us by replying to this email.
```
安装kramdown和rouge：

```bash
    $ gem install kramdown rouge
```

修改*_config.yml*文件：

```ruby
highlighter: rouge
markdown: kramdown
kramdown:
  input: GFM
  extensions:
    - autolink
    - footnotes
    - smart
  syntax_highlighter: rouge
```
配置rouge后，需要引入一个css文件，可以使用

```ruby
$ rougify help style

usage: rougify style [<theme-name>] [<options>]

Print CSS styles for the given theme.  Extra options are
passed to the theme.  Theme defaults to thankful_eyes.

options:
  --scope       (default: .highlight) a css selector to scope by

available themes:
  base16, base16.dark, base16.monokai, base16.monokai.light, base16.solarized, base16.solarized.dark, colorful, github, molokai, monokai, monokai.sublime, thankful_eyes
```
查看目前可用的高亮主题，例如 monokai.sublime 主题，使用

```ruby
rougify style monokai.sublime > rouge_monkai.css 
```
把这个css拷贝到自己的css处就好了
>所有Rouge的支持语法称呼参考这里: Rouge: [List of supported languages and lexers](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers)

这样的方式还是比较直观的，但是可能会存在一个坑，先看看 Rouge 对 Backtick 代码块生成的 html 代码

```html
<div class="highlighter-rouge">
    <pre class="highlight">
        <code>
            ...
        </code>
    </pre>
</div>
```
可见 Rouge 应用了一个 highlighter-rouge 类的 <div></div> 以及 highlight 类的 <pre></pre>，再看看 rougify 之前生成的 CSS 文件的头部

```css
.highlight table td { padding: 5px; }
.highlight table pre { margin: 0; }
.highlight .gh {
  color: #999999;
}
```
并没有对 highlighter-rouge 类的 <div></div> 或是 highlight 类的 <pre></pre> 进行样式说明，这直接可能导致代码块的 background-color 属性被覆盖（如 Bootstrap），若是恰好被覆盖为浅色背景，对于 monokai 这样的前景色为浅色的高亮主题，可能会导致代码块看不清楚，坑爹啊！要修复这个问题，需要对之前生成的 css 做一点小改动，加入一行

```css
pre[class='highlight'] {background-color:#000000;}
```
这样来设置 highlight 类的 <pre></pre> 标签的背景为黑色。

>可以参考这个站[https://github.com/richleland/pygments-css](https://github.com/richleland/pygments-css)提供了很多样式，只需把*codehilite*替换成*highlight*即可
>参考[http://platinhom.github.io/2016/02/04/update-github-rouge/](http://platinhom.github.io/2016/02/04/update-github-rouge/)
>参考[http://blog.javachen.com/2015/06/30/jekyll-kramdown-config.html](http://blog.javachen.com/2015/06/30/jekyll-kramdown-config.html)
>参考[https://oncemore2020.github.io/blog/upgrade-jekyll/](https://oncemore2020.github.io/blog/upgrade-jekyll/)

## 本地测试问题
之前因为gem源问题，导致本地无法使用jekyll构建，就使用了比较麻烦的方法，每次都发布到GitHub上去，通过查看结果来看是否构建成功，结果是悲剧的，收到了好多失败的邮件，下面是其中一封：

```ruby
    The page build failed with the following error:
     
    Page build failed. For more information, see https://help.github.com/articles/troubleshooting-github-pages-build-failures.
     
    GitHub Pages was recently upgraded to Jekyll 3.0. It may help to confirm you're using the correct dependencies:
     
    https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0
     
    If you have any questions you can contact us by replying to this email.
```
问题是还没有具体的错误，就跑到`https://help.github.com/articles/troubleshooting-github-pages-build-failures`链接里看帮助文档，可排列了好多问题，哪知道是哪个，再后来就各种查，原来还有个测试工具`Travis CI`，是一款新兴的开源持续集成构建项目，可以使用GitHub账户登录，选择构建的项目，从`Travis CI`提供的监控页面中就可以看到每次push操作后的构建过程，超级详细，就是那个过程稍微慢了那么点。

想要使用[`Travis-CI`](https://travis-ci.org/)，必须有GitHub账号和GitHub项目，然后在根目录下创建`.travis.yml`文件

```bash
    $ touch .travis.yml
```
配置如下：

```ruby
    language: ruby
    script: "bundle exec jekyll build"
```
>travis-ci使用如下：

![1]({{ site.BASE_PATH }}/assets/img/2016-04-14-my-github-problem-img0.png)

![2]({{ site.BASE_PATH }}/assets/img/2016-04-14-my-github-problem-img3.png)
