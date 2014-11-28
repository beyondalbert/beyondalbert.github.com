---
layout: post
title: "nginx + puma部署rails开发环境记录"
description: ""
category: rails笔记 
tags: [rails, ruby]
---
{% include JB/setup %}

##### 环境

ubuntu 12.04 rails 4.0 ruby 2.0 nginx 1.4.5 puma 2.7.1

##### 部署步骤：

##### 1、安装nginx

* 使用nginx.org的官方repo：

```shell
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ABF5BD827BD9BF62
```

* 把下面一行添加到/etc/apt/sources.list中：

```shell
deb http://nginx.org/packages/ubuntu/ precise nginx
```

* 如果之前系统中安装过nginx，则先把它删除

```shell
sudo apt-get purge nginx*
```

* 安装nginx：

```shell
sudo apt-get update
sudo apt-get install nginx
```

* 安装成功后可以通过如下的命令验证：

```shell
nginx -v
result: nginx version: nginx/1.4.5
```

* 启nginx、停止nginx和重启nginx：

```shell
sudo service nginx start
sudo service nginx stop
sudo service nginx restart
```

##### 2、配置nginx

* 删除默认的配置文件default.conf

```shell
sudo rm /etc/nginx/conf.d/default.conf
```
* 添加应用配置文件

```shell
upstream app-name {
  server unix:///path/to/your/app/root/tmp/sockets/production.socket;
}

server {
  listen 80;
  server_name my_app_url.com; # change to match your URL
  root /path/to/your/project/public; # I assume your app is located at that location

  location / {
    proxy_pass http://app-name; # match the name of upstream directive which is defined above
      proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location ~* ^/assets/ {
    # Per RFC2616 - 1 year maximum expiry
    expires 1y;
    add_header Cache-Control public;

    # Some browsers still send conditional-GET requests if there's a
    # Last-Modified header or an ETag header even if they haven't
    # reached the expiry date sent in the Expires header.
    add_header Last-Modified "";
    add_header ETag "";
    break;
  }
}
```

* 重启nginx，使配置生效

##### 3、用puma启动应用，前提是你可以通过rails s -e production把应用启动

* Gemfile中添加puma

```ruby
gem 'puma'
```
这时，用rails s启动时，puma会替换WEBrick

* 配置应用的puma配置文件

```ruby
#!/usr/bin/env puma

# start puma with:
# bundle exec puma -e production -C path/to/your/app/config/puma.rb

application_path = '/path/to/your/app'
railsenv = 'production'
directory application_path
environment railsenv
daemonize true
pidfile "#{application_path}/tmp/pids/puma-#{railsenv}.pid"
state_path "#{application_path}/tmp/pids/puma-#{railsenv}.state"
stdout_redirect
"#{application_path}/log/puma-#{railsenv}.stdout.log"
"#{application_path}/log/puma-#{railsenv}.stderr.log"
threads 0, 16
bind "unix://#{application_path}/tmp/sockets/#{railsenv}.socket"
```

* 在应用中添加/tmp/pids/和/tmp/sockets/两个文件夹，然后通过一下命令启动应用：

```shell
bundle exec puma -e production -C path/to/your/app/config/puma.rb
```

* 确定puma是否起来

```shell
ps aux | grep puma
login-name    4548  0.3  3.5 367436 63952 ?        Sl   22:44   0:00 /home/albert/.rvm/gems/ruby-2.0.0-p353/bin/puma  
```
如果没有这个puma的进程，说明puma没有启动，需要排查为什么没有启动,一般有可能是/tmp/pids/这个文件夹没有创建，导致pid文件没有生成

##### done

经过以上的配置，我们应该可以通过域名或ip访问到我们的应用，如果没有，请检查以下几项(不限于这些)：

* bind在nginx中的配置和puma中的配置是否一致
