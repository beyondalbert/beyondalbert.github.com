---
layout: post
title: "rails 中配置数据的放置位置探讨"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### web项目的用户配置数据定义

* 程序中某些常量的值
* 某些类型数据，比如像项目类型（公开项目、私有项目等类别）
* 等等

### 如何在rails项目中放置这些数据

#### 1、放置在数据库中

专门建一个配置表，用于存放这些用户配置数据。使用时，只要从数据库中读取即可。

优点： 扩展方便，当你需要增加一个配置量时，只要增加一条数据库记录即可。

缺点： 读取的速度偏慢，特别是当数据库和app不在同一台server上时，需要加上网络延时时间。

#### 2、放置在配置文件中

a、在config/下新建一个配置文件： app_config.yml

```ruby
# 放置三个运行模式下都需要的配置
defaults: &DEFAULTS
  # 示例数据
  project_type:
    pub: "公开项目"
    pri: "私有项目"

# 放置在开发环境下的配置
development:
  <<: *DEFAULTS

# 放置在测试环境下的配置
test:
  <<: *DEFAULTS

# 放置在生产环境下的配置
production:
  <<: *DEFAULTS
```

b、在app启动时，加载这个配置文件

新建文件 initialiers/load_config.rb， 内容示例如下：

```ruby
APP_CONFIG = YAML.load_file("#{Rails.root}/config/app_config.yml")[Rails.env]
```

c、在项目代码中使用该配置数据

```ruby
APP_CONFIG['project_type']['pub']    # => "公开项目"
```

优点： 获取配置数据快，直接在内存中读取，结构也比较清晰，各个环境下能够单独配置不同的数据
缺点： 一旦需要添加配置数据，需要服务重启

### 综合看下来，配置数据还是建议放在配置文件中，维护方便，效率又高