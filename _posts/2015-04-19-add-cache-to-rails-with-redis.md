---
layout: post
title: "rails中使用redis + cache，提高访问速度"
description: ""
category: rails笔记
tags: [rails, 性能调优]
---
{% include JB/setup %}

### rails中使用redis + cache，提高访问速度

redis可以作为缓存服务器，配合rails的cache，可以提升网站的速度

#### 安装redis

##### MAC下安装

使用homebrew

```shell
brew install redis
```

安装完成后，会在控制台上显示设置开机启动的方法及启动redis的方法，重新复制在下面：

```shell
# 设置开机启动
ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist​

# 如何启动服务
redis-server /usr/local/etc/redis.conf
```

##### debian下安装

需要直接下载源码，自己编译安装：

```shell
# 下载源码
wget http://download.redis.io/redis-stable.tar.gz

# 解压源码压缩包
tar xvzf redis-stable.tar.gz

# 进入到解压目录，进行编译安装
sudo make install

# 配置redis
sudo ./utils/install_server.sh

# 输入命令后，可以一路默认，都直接回车，最后配置完成后显示一下配置内容
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
```

安装配置完毕后，我们就可以方便的使用一下命令启动、停止和重启redis服务

```shell
sudo service redis_6379 start[stop restart]
```

#### 监控redis

##### redis提供了命令行命令，供我们方便的监控redis

```shell
# 连接redis服务
redis-cli

# 连接到redis后，可以通过一下命令查看redis运行的数据
# 显示所有的运行数据
info

# 只显示redis的状态数据
info stats
```

作为缓存服务器，我们需要关心以下两个指标：

```shell
# 连接redis，输入info stats 查看以下两个指标：
keyspace_hits:28 # cache命中次数
keyspace_misses:10 # cache没有命中次数

# 通过以上两个数据就可以很方便的计算出cache得命中率： 28 * 100 / (28 + 10) = 73.7%
```

##### 通过第三方工具redis-stat

git 源码地址：

https://github.com/junegunn/redis-stat

用ruby写得一个简单的redis监控工具，具体使用方法，可以参看readme页面

#### 在rails中配置redis

##### redis配置

在正式开始使用redis之前，建议进行如下的配置：

```shell
# 打开redis的config文件：MAC（/usr/local/etc/redis.conf），debian（/etc/redis/6379.conf）

# 限定最多使用的内存
maxmemory 1536mb

# 设置达到内存使用上限后，cache失效的策略
maxmemory-policy allkeys-lru
```

##### 在config/application.rb中添加一下配置项

```ruby
config.cache_store = :redis_store, "redis://localhost:6379/0/cache"
```

重启服务后，你就可以使用redis来存储我们的cache了，真的很简单。

ps：在development下，你需要在development.rb中打开perform_caching配置，不然cache是不会生效的

```ruby
config.action_controller.perform_caching = true
```

#### rails中进行cache开发

TODO
这边帖子中讲的蛮好，就直接先放上链接吧：
https://ruby-china.org/topics/21488

#### 参考链接

http://redis.io/topics/quickstart
http://stackoverflow.com/questions/5636299/rails3-caching-in-development-mode-with-rails-cache-fetch
https://ruby-china.org/topics/21488
https://ruby-china.org/topics/22761