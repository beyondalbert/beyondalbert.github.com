---
layout: post
title: "rails 中使用 jstree 使用记录"
description: ""
category: 
tags: []
---
{% include JB/setup %}

环境： 基于jstree 3.0.0，rails： 4.1.4

### jsTree介绍
1. Jquery插件
2. 在网页中生成具有交互功能的树状结构
3. 开源、免费
4. 容易扩展、配置
5. 支持HTML和JSON格式的数据源
6. 支持AJAX动态获取数据

### 在rails中配置
1. 在http://www.jstree.com/ 主页下载jstree插件源码
2. 把js, css及图片资源放到对应的rails项目目录中

```javascript
// 32px.png 40px.png 和 throbber.gif 放到images目录
// src/themes/style.css 重命名为jstree-style.css.scss放到stylesheets目录
// src目录下的js文件： jstree.js放入javascripts文件夹
// 其余的都是jstree的插件，有需要就放，没有就可以不放进来：jstree.checkbox.js jstree.contextmenu.js jstree.dnd.js jstree.search.js jstree.sort.js jstree.state.js jstree.types.js jstree.unique.js jstree.wholerow.js
// 在application.js中设置js文件的加载顺序，不然会出现js错误。jstree需要在插件之前，实例如下：
// //=require jquery
// //=require jquery_ujs
// //=require jquery-ui
// //=require jstree.vakata
// //=require jstree
// //=require jstree.contextmenu
// .
// .
// .
// //= require_tree
```
ps: 对于jstree.vakata.js文件，我这边的实践是，只有引入了这个文件，jstree的有些功能才能正常进行，但是作者也在这里有过说明，说是不需要引入： https://github.com/vakata/jstree/issues/123

### 一棵简单的树，有html预设置的节点和少量js组成：
```html
<!--views/home/simple_tree.erb-->
<div id="mytree-html">
  <ul>
    <li data-jstree='{"type": "fold"}'>文件夹1</li>
    <li data-jstree='{"type": "fold"}'>文件夹2
      <ul>
        <li data-jstree='{"icon": "/assets/file.png"}'>文件1</li>
      </ul>
    </li>
  </ul>
</div>

<script>
  $('#mytree-html').bind('loaded.jstree', function(e, data) {
    // 当jstree被load完成后，打开所有的节点
    data.instance.open_all();
  });
  $('#mytree-html').jstree({
    "types": {
      "fold": {
        "icon": ".<%= asset_path('fold.png') %>"
      },
      "file": {
        "icon": ".<%= asset_path('file.png') %>"
      }
    },
    'plugins': ['html_data', 'types', 'themes']
  });
</script>
```

### 一颗复杂的树，通过后台的json数据自动生成，同时添加了可以自由的拖拽节点，节点右键菜单功能

#### 通过JSON数据生成树的基本设置：

```javascript
  $('#mytree').jstree({
    'core': {
      // check_callbackc参数用于Contextmenu 和Drag & drop 插件，设置true后才能起作用
      'check_callback': true,
      'data': {
        // 后台JSON数据的地址，当jstree载入时，会自动从这个地址获取json数据，然后生成树状结构
        'url': '/home/tree_data',
        'data': function(node) {
          return {node_id: node.id}
        }
      }
    },
    // types插件设置，但是在我的实践中，这个怎么也没法起效，有待进一步研究
    'types': {
      '#': {
        'valid_children': ['fold', 'file']
      },
      'fold': {
        'icon': "<%= asset_path('fold.png') %>",
        'valid_children': ['fold', 'file']
      },
      'file': {
        'icon': "<%= asset_path('file.png') %>",
        'valid_children': []
      }
    },
    'plugins': ['types', 'dnd', 'contextmenu', 'wholerow'],
    // 右键自定义菜单设置，customMenu是一个函数，下面马上就会讲到
    'contextmenu': {
      'items': customMenu
    }
  });
```

#### 右键自定义菜单处理：

```javascript
  // 自定义右键菜单
  function customMenu(node) {
    var items = {
      'createSubFold': {
        'label': '新建子目录',
        'action': function(obj) {
          var inst = $.jstree.reference(obj.reference);
          var node = inst.get_node(obj.reference);
          inst.create_node(node, {}, 'last', function(new_node) {
            setTimeout(function() {inst.edit(new_node);}, 0);
          });
        }
      },
      'createfile': {
        'label': '新建文件',
        'action': function(obj) {
          var inst = $.jstree.reference(obj.reference);
          var node = inst.get_node(obj.reference);
          $.ajax({
            url: '/home/new_file',
            data: {fold_id: node.id},
            success: function(data, textStatus, jqXHR) {
              console.log("new file success!");
            }
          });
        }
      },
      'rename': {
        'label': '重命名',
        'action': function(obj) {
          var inst = $.jstree.reference(obj.reference);
          var node = inst.get_node(obj.reference);
          inst.edit(node);
        }
      },
      'edit': {
        'label': '编辑',
        'submenu': {
          'cut': {
            'label': '剪切',
            'action': function(obj) {}
          },
          'copy': {
            'label': '复制',
            'action': function(obj) {}
          },
          'paste': {
            'label': '粘帖',
            'action': function(obj) {}
          },
        }
      },
      'destroy': {
        'label': '删除',
        'action': function(obj) {
          var inst = $.jstree.reference(obj.reference);
          var node = inst.get_node(obj.reference);
          $.ajax({
            url: '/home/destroy_node',
            type: 'DELETE',
            data: {id: node.id}
          });
        }
      }
    };
    // 当树的节点是文件时，右键菜单没有新建子文件夹
    // 通过json格式传过来的li_attr或a_attr我们可以轻松的实现不同的节点，拥有不同的右键菜单
    if (node.li_attr.class == 'file') {
      delete items.createSubFold;
    }
    return items;
  }
```

#### 处理树的节点移动事件

```javascript
  $('#mytree').bind('move_node.jstree', function(e, data) {
    // 当页面树的节点位置发生变化后，同时需要更新后端存储的树的结构
    $.ajax({
      url: '/home/move_node',
      type: 'POST',
      data: {id: data.node.id, old_parent: data.old_parent, old_position: data.old_position, parent: data.parent, position: data.position},
      success: function(data, textStatus, jqXHR) {
        console.log('move node successfully!');
      }
    });
  });
```

#### 处理树中节点的左键单击选中事件

```javascript
  // 实现的功能，通过左键点击节点，可以显示节点的详细信息
  $('#mytree').bind('select_node.jstree', function(e, data) {
    // 只有左键选中node才触发ajax操作
    if (data.event.which == 1) {
      if (data.node.li_attr.class == 'file') {
        $.ajax({
          url: '/home/show_file',
          data: {id: data.node.id}
        });
      }
      else {
        $.ajax({
          url: '/home/show_fold',
          data: {id: data.node.id}
        });
      }
    }
  });
```

#### 处理树中节点的重命名事件

```javascript
  // 前端重命名后，需要同时处理后端的重命名，同时，创建一个子目录时，也是调用这个callback
  $('#mytree').bind('rename_node.jstree', function(e, data) {
    $.ajax({
      url: '/home/node_rename',
      type: 'POST',
      data: {id: data.node.id, new_name: data.node.text},
      success: function(data, textStatus, jqXHR) {
        console.log("rename successfully!");
      }
    });
  });
```

#### 实现双击打开或关闭当前节点

```javascript
$('#mytree').delegate('a', 'dbclick', function(e) {
  $('#mytree').jstree('toggle_node', this);
  e.preventDefault();
  return false;
};
```

### 刷新局部的jstree

```javascritp
$('#jstree-demo').jstree(true).refresh_node('#' + node_id);
```

### 刷新整颗树

```javascript
$('#jstree-demo').jstree(true).refresh();
```

使用效果参见：https://github.com/beyondalbert/jstree-demo

参考：
http://www.jstree.com/