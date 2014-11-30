---
layout: post
title: "通过webmock mock Rspec中测试的外部依赖"
description: "通过webmock mock Rspec中测试的外部依赖"
category: rails笔记
tags: [rails, ruby, test]
---
{% include JB/setup %}

环境版本： webmock: 1.20.4 rspec-rails: 3.0.2

### 为什么要mock外部的api依赖

1. 外部依赖不可控，会导致我们自己的测试结果也不可控
2. 外部依赖的api会使我们的测试变慢
3. 不会产生过多的无用外部数据

### webmock整合到Rspec的配置

1. 在Gemfile中添加gem

```ruby
group :test do
     gem 'webmock'
end
```

2. 在spec_helper.rb中添加

```ruby
require 'webmock/rspec'
```

ps: 默认情况下，只要添加了webmock，当前应用所有的外部依赖都被阻断了，除非你在spec_helper.rb中打开

```ruby
# 打开所有的外部链接
WebMock.allow_net_connect!

# 或者只允许部分外部链接，并且允许正则匹配
WebMock.disable_net_connect!(:allow => [/example.org/, /google/])
```

### 直接mock一个特定的外部链接

直接在 it 语句中使用：

```ruby
it "should return project hash" do
    stub_request(:get, "http://www.example.com/api/projects/1.json").to_return(:body => {id: 1, name: "test project"}.to_json)实力
    res = JSON.parse(Net::HTTP.get(URI('http://www.example.com/api/projects/1.json')))
    expect(res).to eq({id: 1, name: "test project"})
end
```

更多的mock实例可以参加github：
https://github.com/bblimke/webmock

### 通过rack应用，在模拟一个外部的api服务依赖，把所有api服务全部发送到自己假造的rack服务上

应用场景：你的应用非常多的依赖于一个外部的api服务，通过上面的简单mock，会重复写多个mock，并且测试代码也会变得分散无法很好的维护。

1. 在每个测试开始前，设置mock的api到特定的rack server（这里用sinatra作为rack server）
先在Gemfile中添加sinatra这个gem：

```ruby
group :development, :test do
    gem 'sinatra'
end
```
在spec_helper.rb中添加如下的代码：

```ruby
config.before(:each) do
    stub_request(:any, /example.com/).to_rack(FakeServer)
end
```

2. 在spec/support/下添加rack应用： fake_server.rb

```ruby
# spec/support/fake_server.rb
require 'sinatra/base'
class FakeApiServer < Sinatra::Base
    get '/api/projects/1.json' do
        content_type :json
        status 200
        {id: 1, name: "test project"}.to_json
    end
end
```

3. 在测试中的使用：

```ruby
it "should return project hash" do
    # 下面的url会自动的发送的我们的fake server上，然后返回fake server上设置的返回值
    res = JSON.parse(Net::HTTP.get(URI('http://www.example.com/api/projects/1.json')))
    expect(res).to eq({id: 1, name: "test project"})
end
```

参考： 
http://robots.thoughtbot.com/how-to-stub-external-services-in-tests
https://github.com/bblimke/webmock
