---
layout: post
title: "使用logrotate分割rails log"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 使用logrotate分割rails log

生产环境中得log如果不进行切分管理，那么其随着时间的推移，log文件的大小会越来越大，不便于日志管理

服务器环境：debian

#### 如何配置logrotate 对rails log进行切分

通过简单的配置logrtate配置文件，就可以轻松的实现log的分割。打开/etc/logrotate.conf，添加以下配置到该文件的末尾

```ruby
/path/to/your/rails/log/path/*.log {
  daily
  missingok
  rotate 7
  compress
  delaycompress
  notifempty
  copytruncate
  dateext
}
```

#### logrotate配置项解释

* daily: 每天都切分log，你也可以设置为：weekly（每周切分）、monthly（每月切分）
* missingok: 如果log文件不存在就忽略不切分
* rotate: 设置log的保存份数，如果希望不设置，则保留所有的切分log
* compress: 对切分的log进行gzip压缩
* delaycompress: 最近的切分的一次log不进行压缩，就是延迟一次压缩
* notifempty: 如果log为空的话，不进行切分
* copytruncate: 切分的时候，采用copy log中得内容，然后再清空原先的log文件。这样可以保证rails 的log文件一直存在，如果不怎么设置，你必须每次都需要重启rails 服务
* dateext: 设置后，会再切分的log文件后加上日期后缀

未设置的logrotate配置

* size 1k: 当log大小达到1k大小的时候进行log切分
* postrotate: 切分log后，执行指定的脚本

```shell
/log/path/*.log {
  postrotate
    /path/to/script.sh
  endscript
}
```

* maxage: 设置最多保存的log份数

#### 设置后的测试

运行以下命令进行测试：

```shell
sudo /usr/sbin/logrotate -f /etc/logrotate.conf
```

如果log文件夹中只有production.log文件，执行一次以上shell命令后，就会多出切分的文件。

#### 参考链接
https://gorails.com/guides/rotating-rails-production-logs-with-logrotate
http://www.thegeekstuff.com/2010/07/logrotate-examples/
