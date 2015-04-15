---
layout: post
title: "使用god监控rails unicorn进程"
description: ""
category: rails笔记
tags: [rails, 运维部署]
---
{% include JB/setup %}

### 使用god监控rails unicorn进程

God是一个ruby的进程监控框架，可以方便的对你的rails 服务进程进行监控

#### 安装

```ruby
gem install god
```

### 配置rails监控配置文件

新建配置文件：RAILS_ROOT/config/unicorn.god

内容如下：

```ruby
# 获取rails根目录，方便维护
RAILS_ROOT = File.dirname(File.dirname(__FILE__))

# 设置God发送邮件的配置
God::Contacts::Email.defaults do |d|
  d.from_email = 'god@example.com'
  d.from_name = 'God'
  d.delivery_method = :sendmail
end
# 设置发送邮件的联系人
God.contact(:email) do |c|
  c.name = 'Dev Team'
  c.group = 'devteam'
  c.to_email = 'devteam@example.com'
end

# 监控代码块
God.watch do |w|
  # 设置自己环境下的unicorn pid文件路径
  pid_file = File.join(RAILS_ROOT, "tmp/unicorn.pid")

  w.name = "unicorn"
  w.dir = RAILS_ROOT
  w.interval = 60.seconds

  # start stop restart 需要替换成自己的启动 停止和重启命令
  w.start = "RAILS_ENV=production unicorn -c #{RAILS_ROOT}/config/unicorn.rb -D"
  w.stop = "kill -s QUIT $(cat #{pid_file})"
  w.restart = "kill -s HUP $(cat #{pid_file})"

  w.start_grace = 20.seconds
  w.restart_grace = 20.seconds
  w.pid_file = pid_file

  # 清除pid文件，不然无法重启
  w.behavior(:clean_pid_file)

  # When to start?
  w.start_if do |start|
    start.condition(:process_running) do |c|
      # We want to check if deamon is running every ten seconds
      # and start it if itsn't running
      c.interval = 10.seconds
      c.running = false
    end
  end

  # When to restart a running deamon?
  w.restart_if do |restart|
    restart.condition(:memory_usage) do |c|
      # Pick five memory usage at different times
      # if three of them are above memory limit (100Mb)
      # then we restart the deamon
      c.above = 100.megabytes
      c.times = [3, 5]
    end

    restart.condition(:cpu_usage) do |c|
      # Restart deamon if cpu usage goes
      # above 90% at least five times
      c.above = 90.percent
      c.times = 5
    end
  end

  w.lifecycle do |on|
    # Handle edge cases where deamon
    # can't start for some reason
    on.condition(:flapping) do |c|
      c.to_state = [:start, :restart] # If God tries to start or restart
      c.times = 5                     # five times
      c.within = 5.minute             # within five minutes
      c.transition = :unmonitored     # we want to stop monitoring
      c.retry_in = 10.minutes         # for 10 minutes and monitor again
      c.retry_times = 5               # we'll loop over this five times
      c.retry_within = 2.hours        # and give up if flapping occured five times in two hours
    end
  end

  # 每次进程挂掉时，发送邮件
  w.transition(:up, :start) do |on|
    on.condition(:process_exits) do |c|
      c.notify = 'devteam'
    end
  end
end
```

#### God的常用命令

```ruby
# 查看god的监控状态
god status

# 如何启动god
god -c config/unicorn.god

# 带上参数 -D 将不会进程化
god -c config/unicorn.god -D

# 查看监控log
god log unicorn

# 终止所有监控
god terminate

# 终止某一个监控
god stop unicorn

# 停止god，而不影响应用进程
god quit
```

#### 配置过程中遇到的问题

1、无法启动god，用-D启动，报如下的错误：

```ruby
ERROR: Socket drbunix:///tmp/god.17165.sock already in use by another instance of god
```
直接用god terminatem命令干掉所有god，再启动god就可以

2、本地Mac可以正常启动，但是到了线上debian环境，出现如下错误：

```ruby
ERROR: Condition 'God::Conditions::ProcessExits' requires an event system but none has been loaded
```

需要用sudo权限才能load event system，因此，我这边采用了rvmsudo god 就可以正常的启动了

3、都配置好后，发现无法发报警邮件

检查一下你的服务器的邮件服务，check一下默认的邮件端口25是否打开等

```shell
# 用命令行测试一下，是否可以发出邮件
mail -s "hello" test@example.com

# 回车后，输入邮件正文，以 "." 号结束，回车再回车
Hi,
this is a test
.
Cc:
```

#### 参考链接

http://www.synbioz.com/blog/monitoring_server_processes_with_god
https://github.com/mojombo/god/issues/99





