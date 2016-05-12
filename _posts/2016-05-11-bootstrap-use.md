---
layout: postlayout
title: Bootstrap与jQuery使用
description: 数据传输中保证数据安全，使用加密算法对数据进行加密，尤其是互联网环境下对外暴露接口，比如restfull接口，数据的安全性更是重中之重
thumbimg: 1346208288725.jpg
categories: [Bootstrap]
tags: [Bootstrap,jQuery]
---
这里只是简单的介绍下自己使用Bootstrap过程中的一些方式，觉得Bootstrap在移动设备和扁平化处理方面看起来简约大方，结合jQuery使用，其强大之处也显而易见。

## json数据format

将页面json或者object数据格式化输出
依赖：`jQuery`
来源：`http://www.sharejs.com/codes/javascript/5452`，在此基础上进行修改
```javascript
!function($) {
	"use strict";
	//在HTML中使用<br>代替\r\n也可
	$.formatJson = function(json, options) {
		var reg = null,
        formatted = '',
        pad = 0,
        PADDING = '    '; // 也可以使用 '\t'代替

      // 属性设置-初始化
      options = options || {};
      // 如果冒号“:”后面存在花括号“{”、中括号“[”，即":{"或":["这种，是否需要换行
      options.newlineAfterColonIfBeforeBraceOrBracket = (options.newlineAfterColonIfBeforeBraceOrBracket === true) ? true : false;
      // 判断冒号后是否添加空格
      options.spaceAfterColon = (options.spaceAfterColon === false) ? false : true;
      
      // 判断格式化json是否为string类型，否则转换
      if (typeof json !== 'string') {
        json = JSON.stringify(json);
      }
      
      // 格式化解析去除多余的空格
      json = JSON.parse(json);
      json = JSON.stringify(json);

      // 在花括号“{}”前后添加换行回车符
      reg = /([\{\}])/g;
      json = json.replace(reg, '\r\n$1\r\n');

      // 在方括号“[]”前后添加换行回车符
      reg = /([\[\]])/g;
      json = json.replace(reg, '\r\n$1\r\n');
      
      // 在逗号“,”后添加回车换行符
      reg = /(\,)/g;
      json = json.replace(reg, '$1\r\n');
      
      // 去掉包含多个的回车换行
      reg = /(\r\n\r\n)/g;
      json = json.replace(reg, '\r\n');
      
      // 去掉逗号“,”前面的回车换行
      reg = /\r\n\,/g;
      json = json.replace(reg, ',');

      if (!options.newlineAfterColonIfBeforeBraceOrBracket) {     
        reg = /\:\r\n\{/g;
        json = json.replace(reg, ':{');
        reg = /\:\r\n\[/g;
        json = json.replace(reg, ':[');
      }
      if (options.spaceAfterColon) {      
        reg = /\:/g;
        json = json.replace(reg, ': ');
      }
      reg = /^(\r\n)?/;
      json = json.replace(reg, "");

      $.each(json.split('\r\n'), function(index, node) {
        var i = 0,
          indent = 0,
          padding = '';
        
        if (node.match(/\{$/) || node.match(/\[$/)) {
          indent = 1;
        } else if (node.match(/\}/) || node.match(/\]/)) {
          if (pad !== 0) {
            pad -= 1;
          }
        } else {
          indent = 0;
        }
      
        for (i = 0; i < pad; i++) {
          padding += PADDING;
        }
      
        formatted += padding + node + '\r\n';
        pad += indent;
      });
      
      return formatted;
	};
	
	$.ajaxSubmit = function(url) {
		$.ajax({
	        url: url,
	        type: 'post',
	        data: $('#inputForm').serialize(),
	        dataType: 'json',
	        timeout: 30000,
	        error: function () {
	            alert('处理超时或系统处理错误');
	        },
	        success: function (data) {
	        	$('#responseHead').empty();
	        	$('#responseBody').empty();
	        	$('#responseHead').append(data.head);
	        	$('#responseBody').append($.formatJson(data.body));
	        }
	    });
	};
}(window.jQuery);
```

## 导航栏

简单的使用导航菜单和导航条
依赖：`jQuery` `Bootstrap[bootstrap-cerulean.min.css]`

### 左侧导航栏

```html
<div class="sidebar-nav">
    <div class="nav-canvas">
        <div class="nav-sm nav nav-stacked">

        </div>
        <ul class="nav nav-pills nav-stacked main-menu">
            <li><a class="" href=""><i class="glyphicon glyphicon-lock"></i><span> 登录</span></a></li>
            <li class="accordions">
                <a><i class="glyphicon glyphicon-align-justify"></i><span> 账户</span></a>
                <ul class="nav nav-pills nav-stacked">
                    <li><a class="" href="">注册</a></li>
                    <li><a class="" href="">开通</a></li>
                    <li><a class="" href="">状态查询</a></li>
                </ul>
            </li>
            <li class="accordions">
                <a><i class="glyphicon glyphicon-align-justify"></i><span> 资金</span></a>
                <ul class="nav nav-pills nav-stacked">
                    <li><a class="" href="">派发</a></li>
                    <li><a class="" href="">查询</a></li>
                    <li><a class="" href="">充值</a></li>
                    <li><a class="" href="">账户资金总览</a></li>
                </ul>
            </li>
            <li class="accordions">
                <a><i class="glyphicon glyphicon-align-justify"></i><span> 测试</span></a>
                <ul class="nav nav-pills nav-stacked">
                    <li><a class="" href="">测试1</a></li>
                    <li><a class="" href="">测试2</a></li>
                    <li><a class="" href="">测试3</a></li>
                    <li><a class="" href="">测试4</a></li>
                    <li><a class="" href="">测试5</a></li>
                </ul>
            </li>
            <li class="accordions">
                <a><i class="glyphicon glyphicon-align-justify"></i><span> 游戏</span></a>
                <ul class="nav nav-pills nav-stacked">
                    <li><a class="" href="">查询</a></li>
                    <li><a class="" href="">查询</a></li>
                    <li><a class="" href="">查询</a></li>
                    <li><a class="" href="">查询</a></li>
                </ul>
            </li>
            <li class="accordions">
                <a><i class="glyphicon glyphicon-align-justify"></i><span> 推送消息查询</span></a>
                <ul class="nav nav-pills nav-stacked">
                    <li><a class="" href="">消息查询</a></li>
                </ul>
            </li>
        </ul>
    </div>
</div>
```
效果如如下：
![1]({{ site.BASE_PATH }}/assets/img/20160511221919.png)

### 导航条

```html
<div class="col-lg-12 col-sm-12">
	<div class="navbar navbar-default navbar-inner">
		<div class="navbar-header">
		<div class="navbar-brand">
			<p>运营活动系统-模拟器</p>
		</div>
		</div>
		<div class="collapse navbar-collapse">
			<ul class="nav navbar-nav">
				<li class="active"><a href="${ctx}/login/init">登录</a></li>
				<li class="dropdown">
					<a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">账户<span class="caret"></span></a>
					<ul class="dropdown-menu">
						<li><a href="">注册</a></li>
						<li role="separator" class="divider"></li>
						<li><a href="">账号</a></li>
						<li role="separator" class="divider"></li>
						<li><a href="">账户状态查询</a></li>					
					</ul>
				</li>
				<li class="dropdown">
					<a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">资金<span class="caret"></span></a>
					<ul class="dropdown-menu">
						<li><a href="">派发</a></li>
						<li role="separator" class="divider"></li>
						<li><a href="">查询</a></li>
						<li role="separator" class="divider"></li>
						<li><a href="">充值</a></li>	
						<li role="separator" class="divider"></li>				
						<li ><a href="">账户资金总览</a></li>					
					</ul>
				</li>
				<li class="dropdown">
					<a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">游戏<span class="caret"></span></a>
					<ul class="dropdown-menu">
						<li><a href="">查询</a></li>
						<li role="separator" class="divider"></li>
						<li><a href="">查询</a></li>
						<li role="separator" class="divider"></li>
						<li ><a href="">查询</a></li>	
						<li role="separator" class="divider"></li>				
						<li ><a href="">查询</a></li>					
					</ul>
				</li>
				<li class="dropdown">
					<a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">推送消息查询<span class="caret"></span></a>
					<ul class="dropdown-menu">
						<li><a href="">消息查询</a></li>
					</ul>
				</li>
			</ul>
			<ul class="nav navbar-nav navbar-right">
					<li><a href=""><span style="color:red;">注：本模拟器页面中有token字时，需要先登录然后再进行操作，该值不需要手动处理，模拟器会自动获取</span></a></li>
			</ul>
		</div>
	</div>
</div>
```
效果图如下：
![2]({{ site.BASE_PATH }}/assets/img/20160511223021.png)

Bootstrap没有使用太多，就是用了grid、form及各输入组件、可编辑表格，个人喜欢这种简约大方的风格，纯粹兴趣爱好，特此记下。