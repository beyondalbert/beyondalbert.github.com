---
layout: post
title: "Rails 中实现go to top按钮功能"
description: ""
category: rails笔记 
tags: [rails, ruby]
---
{% include JB/setup %}

## rails中回到顶部

回到页面顶部的功能，在web开发中经常需要用到，自己搜索之后，参考了网上的资料，自己总结一下，以备后用

## 1、在layouts页面的底部加入a标签
```ruby
<%= link_to image_tag("go_top.png"), "#", id: "go-top", title: "返回页面顶部" %>
```

## 2、添加如下的js：
```javascript
$(window).scroll(function() {
  var scrollt = document.documentElement.scrollTop + document.body.scrollTop;
  if(scrollt>50){
    $("#go-top").fadeIn(200);
  }else{
    $("#go-top").stop().fadeOut(200);
  }
});

$("#go-top").click(function(){
  $("html,body").animate({scrollTop: "0px"}, 200);
});
```

## 3、添加a标签的css
```css
#go-top {
  display: block;
  width: 20px;
  height: 70px;
  position:fixed;
  bottom: 5px;
  right: 5px;
  border-radius: 2px;
  text-decoration: none;
  display: none;
  background-color: #adadad;
}
```

## 4、去百度或google搜索一个自己满意的图标，比如（ http://www.lanrentuku.com/gif/a/top.html ）这里就有很多可以挑选的，然后根据图标大小，调整一下css的width和height
