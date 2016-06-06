---
layout: postlayout
title: 滑动验证码样例
description: 滑动验证码样例
thumbimg: 1346208288725.jpg
categories: [javascript]
tags: [jquery, javascript]
---
<style>
    html, body, h1 {
        margin: 0;
        padding: 0;
    }
    body {
        background-color: #393939;
        color: #d5d4ff;
        overflow: hidden
    }
    #demo {
        width: 600px;
        margin: 150px auto;
        padding: 10px;
        border: 1px dashed #d5d4ff;
        border-radius: 10px;
        -moz-border-radius: 10px;
        -webkit-border-radius: 10px;
        text-align: left;
    }
</style>
<div id="demo">
  <div id="slider">
    <div id="slider_bg"></div>
    <span id="label">>></span> <span id="labelTip">拖动滑块验证</span> </div>
  <script type="text/javascript" src="{{ site.BASE_PATH }}/assets/js/jquery.slideunlock.js"></script>
  <script>
    $(function () {
        var slider = new SliderUnlock("#slider",{
				successLabelTip : "欢迎访问My Tech Space"	
			},function(){
				alert("验证成功");
        	});
        slider.init();
    })
</script> 
</div>