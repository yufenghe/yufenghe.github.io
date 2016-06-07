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
    .demo {
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
<div class="demo">
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
    });
</script> 
</div>

<style>
*,
*:after,
*:before {
	-webkit-box-sizing: border-box;
	-moz-box-sizing: border-box;
	box-sizing: border-box;
}

.heart {
	    background: url({{ site.BASE_PATH }}/assets/img/web_heart_animation.png);
	    /* background-position: left; */
	    /* background-repeat: no-repeat; */
	    height: 100px;
	    width: 100px;
	    /* cursor: pointer; */
	    position: absolute;
	    /* left:-14px; */
	    /* background-size:2900%; */
}
.heart:hover, .heart:focus{
    background-position: right;
}

@-webkit-keyframes heartBlast {
    0% {
	    background-position: left;
	}
	100% {
	    background-position: right;
	}
}

@keyframes heartBlast {
	0% {
		background-position: left;
	}
	100% {
	    background-position: right;
	}
}

.heartAnimation {
    display: inline-block;
    -webkit-animation-name: heartBlast;
    animation-name: heartBlast;
    -webkit-animation-duration: .8s;
    animation-duration: .8s;
    -webkit-animation-iteration-count: 1;
    animation-iteration-count: 1;
    -webkit-animation-timing-function: steps(28);
    animation-timing-function: steps(28);
    background-position: right;
}

.feed p{font-family: "Microsoft YaHei",'Georgia', Times, Times New Roman, serif; font-size: 25px;}
.feed{clear: both; margin-bottom: :20px; height: 150px; position: relative;}
p{margin: 0px; padding: 0px;}
.likeCount{font-family: 'Georgia', Times, Times New Roman, serif; margin-top: 32px;margin-left: 68px;font-size: 25px;color: #999999,width:100px;height:100px;line-height:100px;}

.clearfix:before,
.clearfix:after {
	content: " ";
	display: table;
}

.clearfix:after {
	clear: both;
}
</style>

## JS特效
<div class="feed">
<p></p>
<div class="heart" id="like1" rel="like"></div> <div class="likeCount" id="likeCount1">14</div>
</div>

<script>
	$(function() {
	$('body').on("click",'.heart',function() {
    	var A=$(this).attr("id");
    	var B=A.split("like");
        var messageID=B[1];
        var C=parseInt($("#likeCount"+messageID).html());
    	$(this).css("background-position","")
        var D=$(this).attr("rel");
       
        if(D === 'like') {      
			$("#likeCount"+messageID).html(C+1);
			$(this).addClass("heartAnimation").attr("rel","unlike");
        }
        else{
			$("#likeCount"+messageID).html(C-1);
			$(this).removeClass("heartAnimation").attr("rel","like");
			$(this).css("background-position","left");
        }
    });
});
</script>	

## 特效2

<link href="{{ site.BASE_PATH }}/assets/css/heart.css" rel="stylesheet">
<div class="heart-wrapper active">
  <div class="heart">
    <div class="tl"></div>
    <div class="tr"></div>
    <div class="bl"></div>
    <div class="br"></div>
  </div>
  <div class="ring"></div>
  <div class="circles"></div>
</div>

<script>
$(function(){
  var debug = /*true ||*/ false;
  var h = document.querySelector('.heart-wrapper');
  
  function toggleActivate(){
    h.classList.toggle('active');
  }

  if(!debug){
    h.addEventListener('click',function(){
      toggleActivate();
    },false);

    // setInterval(toggleActivate,1000);
  }else{
    var elts = Array.prototype.slice.call(h.querySelectorAll(':scope > *'),0);
    var activated = false;
    var animating = false;
    var count = 0;
    var step = 1000;
    
    function setAnim(state){
      elts.forEach(function(elt){
        elt.style.animationPlayState = state;
      });
    }
    
    h.addEventListener('click',function(){
      if (animating) return;
      if (count > 27) {
        h.classList.remove('active');
        count = 0;
        return;
      }
      if (!activated) h.classList.add('active') && (activated = true);
      
      console.log('Step : '+(++count));
      animating = true;
      
      setAnim('running');
      setTimeout(function(){
        setAnim('paused');
        animating = false;
      },step);
    },false);

    setAnim('paused');
    elts.forEach(function(elt){
      elt.style.animationDuration = step/1000*27+'s';
    });
  }
});

</script>