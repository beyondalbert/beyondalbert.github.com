---
layout: post
title: "rails admin 页面开发实践"
description: ""
category: 
tags: [rails, ruby]
---
{% include JB/setup %}

### 创建一个简单的admin dashbord页面

#### 1、生成controller

```shell
rails g controller admin/dashboard index
```

这个将会生成如下的目录结构：

```shell
controllers
  |__admin
       |__dashboard_controller.rb
```

同时在routes.rb文件中会生成如下的路由：

```ruby
namespace :admin do
  get 'dashboard/index'
end
```

#### 2、增加一个admin_controller.rb用于统一控制在admin这个namespace下的页面布局以及权限

代码内容示例如下：

```ruby
class AdminController < ApplicationController
  layout 'admin' # 如有必要，可以设置admin页面单独的layout
  before_filter :require_admin

  private

  def require_admin
    # admin 权限控制代码
  end
end
```

同时，所有admin namespace下的controller都需要继承这个AdminController,比如在/admin/dashboard_controller.rb中，示例代码如下：

```ruby
class Admin::DashboardController < AdminController
  def index
  end
end
```

ps： 如需要创建其他admin权限的页面，就可以仿照dashboard_controller.rb的创建方法，就可以轻松的开发一个属于自己的web后台管理系统