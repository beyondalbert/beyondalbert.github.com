---
layout: post
title: "Rails Rspec 使用简明指南"
description: "Rails Rspec 使用简明指南"
category: rails笔记
tags: [rails, ruby, test]
---
{% include JB/setup %}

基于版本： rspec_rails: 3.0.4 factory_girl_rails: 4.5.0

### Rails中设置

1. Gemfile中添加如下gem，然后bundle install

``` ruby
group :development, :test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
end
```

2. 设置测试数据库

``` ruby
# config/database.yml
test:
  adapter: mysql2
  encoding: utf8
  reconnect: false
  pool: 5
  socket: /var/run/mysqld/mysqld.sock
  database: demo_test
  username: demo
  password: "****"
```

3. 安装rspec

在rails demo的根目录下运行如下的命令：

``` ruby
rails g rspec:install
```

该命令会生成如下的结果：

```shell
create  .rspec
create  spec
create  spec/spec_helper.rb
```

4. 改变rspec的输出，让人更容易看结果

在.rspec文件中加如下代码：

```ruby
--format documentation
```

#### 替换自带的test框架，设置自动生成rspec测试文件

在config/application.rb中添加一下配置

```ruby
  config.generators do |g|
    g.test_framework :rspec,
      :fixtures => true,
      :view_specs => false,
      :helper_specs => true,
      :routing_specs => false,
      :controller_specs => true,
      :request_specs => true
    g.fixture_replacement :factory_girl, :dir => "spec/factories"
  end
```

### Rspec自动生成命令

```ruby
rails g rspec:model project
# 将生成一下文件：
# spec/models/project_spec.rb
# spec/factories/projects.rb

#还能用于一下内容的自动生成：
rails g rspec:controller project
rails g rspec:helper project
# 等等，具体参见：https://www.relishapp.com/rspec/rspec-rails/docs/generators
```

### Model spec demo

```ruby
require "rails_helper"

Rspec.describe Project, :type => :model do
    # model的实例方法测试
    describe "#issue_count" do
        it "返回项目中的任务个数" do
            project = FactoryGirl.create(:project, :with_issues)
            expect(project.issue_count).to eq(3)
        end
    end
    
    # model的类方法测试
    describe ".total_count" do
        it "返回所有项目的数量" do
            expect(Project.total_count).to eq(1)
        end
    end
end
```

### To Be Continue
