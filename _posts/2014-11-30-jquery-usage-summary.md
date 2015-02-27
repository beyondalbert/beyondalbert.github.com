---
layout: post
title: "jquery使用小记"
description: ""
category: 
tags: [js, jquery]
---
{% include JB/setup %}

#### jquery常用接口

##### jquery选择器

```javascript
// 选中所有元素
$("*");
// 根据类名选择元素
$('.class-demo');
// 根据元素名字来选择元素
$('div');
// 根据元素id来选择元素
$('#id-demo');
// 组合选择多种类型的元素
$('#id-demo, div, .class-demo');
// 根据元素的属性来选择元素，更多属性选择器用法见：
// http://api.jquery.com/category/selectors/attribute-selectors/
$("a[href='/test_demo']");
```

##### .resize：当元素大小变化时，触发这个事件，如果绑定到了window元素，则会在浏览器大小变化的时候触发，实例：

``` javascript
$(window).resize(function() {
    console.log("浏览器宽度为：" + $(window).width());
});
```

##### .hover：当鼠标hover到元素上时，然后离开这个元素，触发这个事件，实例：

``` javascript
$('#target-element-id').hover(function() {
    // 悬浮到目标元素上时，触发这个函数中的事件
    console.log("mouse enter target element");
}, function() {
    // 鼠标离开目标元素时，触发这个函数中的事件
    console.log("mouse leave target element");
});
```

##### .click： 当元素被单击时，触发的事件；

##### .dblclick()：当元素被双击时触发的事件，实例：

``` javascript
$("#target-element-id").click(function() {
    console.log("target element is clicked!");
});
$("#target-element-id").dblclick(function() {
    console.log("target element is double clicked!");
});
```

##### .change：当元素的value发生变化的时候，触发的事件，该事件只对input, textarea, select这三种类型的元素value变化会触发，对于select boxes, checkboxes和radio button，这个事件会会立刻触发，一旦值变化，但对于其他的元素，则是要等到元素失去焦点后才能触发，实例：

``` javascript
$('#target-select-id').change(function() {
    var selectedItemValue = $(this).children('option:selected').val();
    console.log("change the value to: " + selectedItemValue);
});
```

##### .addClass .removeClass： 添加和删除目标元素的css类，实例：

``` javascript
$('#target-element-id').addClass("foo");
$('#target-element-id').removeClass("foo");
```

##### .serialize()：把要提交的表单中的输入框的值编码成字符串，使之可以在url中传输；也可以对单个的表单进行编码，实例：

``` javascript
// 一般用于ajax中，设置需要提交的data
// 同时，一般会阻止默认的表单form提交： event.preventDefault();
var submitData = $("#target-form-id").serialize();
```
ps: 表单form中的表单都需要有name属性，不然这个函数不会把表单的值编码到最终的字符串

##### .text() .html()：前者得到目标元素所有子元素的文本内容，或者设置目标元素的文本内容；后者得到目标元素所有子元素的html内容，或者设置目标元素的html内容

##### .val()：获取或设置匹配选择器的元素的值，一般是针对表单中的元素值的获取

html内容：

```html
<div id="target-id">
    <div>test1</div>
    test2
    <input id="input-id" value="test!">
</div>
```

js实例：

```javascript
// 输出 test1 test2
$('#target-id').text();
// 得到
// <div>test1</div>
// test2
$('#target-id').html();
// 设置目标元素的text
$('#target-id').text('test3');
// 设置目标元素的html内容
$('#target-id').html(htmlContent);
// 获取input框的值
$('#input-id').val();
// 设置input框的值
$('#input-id').val();
```

##### data()： 获取或设置元素中的data值，一般用于不能设置value值的元素，通过设置data值，可以给该元素设置value

```html
<div id="demo-id" data-msg="test msg" data-demo="test demo">
```

```javascript
// 输出test msg
$('#demo-id').data('msg')；
// 设置data-msg的值
$('#demo-id').data('msg', 'test change msg');
```

##### .clone()：克隆目标元素以及其所有子元素，默认不会克隆绑定在子元素上的事件，除非参数中设置true，实例：

```javascript
var cloneTargetElement = $('#target-element-id').clone();
// 通常用于复制后，在append到其他的元素
cloneTargetElement.appendTo("#another-element");
// 如需复制子元素上的事件，需要添加true选项
var cloneTargetElement = $('#target-element-id').clone(true);
```

##### .after 在目标元素后添加参数中的元素，添加的元素是目标元素的兄弟元素

##### .appendTo 在目标元素中，添加参数中的元素，添加的元素是目标元素的子元素

##### .append 把目标元素添加到参数中的元素中去，目标元素是参数元素的子元素

实例：

``` javascript
$('#target-id').after(cloneTargetElement);
$('#target-id').append(cloneTargetElement);
cloneTargetElement.appendTo("#target-id");
```

##### .prop：获取或设置dom对象的属性值

##### .attr：获取或设置html页面元素的属性值

实例：

```html
<input id="check-demo" type="checkbox" checked="checked">
```

```javascript
// 返回true
$('#check-demo').prop('checked');
// 返回“checked”
$('#check-demo').attr('checked');
// 设置checkbox为未选中状态
$('#check-demo').prop('checked', false);
```

##### .each：遍历jquery或得的dom对象，实例：

```html
<ul>
    <li>foo</li>
    <li>bar</li>
</ul>
```

```javascript
$('li').each(function(index) {
    console.log(index + ":" + $(this).text());
});
```

ps: 一般如果是jquery获取的一个dom数组，建议用这个方式来遍历，如果是纯js的数组，建议用forEach方式来遍历

```javascript
var arrayDemo = [1, 2, 3];
arrayDemo.forEach(function(element, index, array) {
    console.log("a[" + index + "] = " + element);
});
```

##### .filter： 过滤选中的dom数组，实例：

```javascript
$('li').filter(function(index) {
    return index % 3 === 2
});
```

##### .find：查找当前元素下的所有元素，返回匹配选择器的元素

##### .children：查找当前元素下的第一层子元素，返回匹配选择器的元素

实例：

````javascript
$('#target-id').find(".demo-class");
$('#target-id').children(".demo-class");
````

##### .scrollTo：jquery的一个插件，用于html内容的滚动，demo可以看链接： http://jsfiddle.net/beyondalbert/ofjr2m7j/

##### .remove: 删除一个html元素，实例：

```html
<div class="container">
    <div class="hello">Hello</div>
    <div class="goodbye">Goodbye</div>
</div>
```

```javascript
// 删除选中的元素
$('.hello').remove();
// 删除选中元素中，符合选择器的子元素
$('.container').remove('.goodbye');
```

##### .show() .hide(): 显示和隐藏目标元素

```javascript
$('#target-id').show();
$('#target-id').hide();
// 比较复杂的用法：
// 控制显示的快慢：用“slow”（600）和“fast”（200），或者是数值，以毫秒为单位
$('#target-id').show("slow", function() {
    // 显示结束后，需要处理的逻辑写在这里
});
```

##### .prev(): 获取目标元素的前面一个元素

```javascript
$('#target-id').prev();

// 目标节点之后的兄弟节点选择，以下选择id为target的元素之后的div兄弟节点
$('#target ~ div');
```

##### .before(): 在目标元素的前面插入html

##### .insertBefore(): 把目标元素插入到指定的元素后面

```javascript
$('#target-id').before("<p>test</p>");
$("<p>test</p>").insertBefore('#target-id');
```

##### 给还没有渲染的元素捆绑事件

```javascript
  $(document).on('click', '#target-element', function() {
    console.log('test');
  });
```

























