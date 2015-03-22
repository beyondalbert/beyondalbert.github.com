---
layout: post
title: "在阿里云上部署rails服务：mysql + unicorn + nginx"
description: ""
category: rails笔记
tags: [rails, 运维部署, unicorn, nginx]
---
{% include JB/setup %}

## 在阿里云上部署rails服务：mysql + unicorn + nginx

环境： debian 7

### 云主机初始化设置

#### 添加一个部署服务的用户，而不是一直采用root用户

##### 添加用户组及用户

```shell
  # 添加一个部署用户组
  groupadd deployers
  # 新建一个用户到该用户组
  adduser deployer1 -ingroup deployers
  # 上面的命令会提示你输入密码，和用户信息，密码一定要输，其他的信息就随意，空着也没事
```
##### 用vi（或其他你喜欢的编辑器）打开sudoers文件(/etc/sudoers)，给用户组添加sudo权限，添加以下内容到该文件

```shell
  %sudo ALL=(ALL:ALL) ALL
  # 上面的语句后添加以下行：
  %deployers ALL=(ALL:ALL) ALL
```

##### 为什么要如此设置？

* 出于安全考虑
* 采用用户组的方式，方便部署用户管理
* sudo权限设置，部署用户在部署时需要用到root权限，因此设置。部署用户需要用到root权限时，只要sudo + 命令就可以执行了


#### 设置SSH连接

##### 打开ssh设置文件（/etc/ssh/sshd_config），修改以下配置：

```shell
  # 修改默认的端口，可选范围1024 ~ 65536
  Port 123456

  # 禁止root用户通过ssh 登陆
  PermitRootLogin no
  # 或者更加保险一点，再设置只有指定的用户才可以登陆
  AllowUsers deployer1
```

##### 重启ssh服务，使刚修改的设置生效。
ps：最好保留一个vps的登陆窗口，防止设置错误后，root用户和新建的用户都无法登陆，只能重置服务这个情况发生

```shell
  # 重启ssh服务
  sudo service ssh restart

  # 测试是否已经生效
  # 可以通过端口 123456 登陆，没有设置端口将登陆不了
  ssh -p 123456 deployer1@xxx.xxx.xxx.xxx
  # 验证以下root用户是否可以登陆
  ssh -p 123456 root@xxx.xxx.xxx.xxx
```

##### 设置ssh key登陆，更加安全的选择

###### 查看有没有现成的，如果有就不用再重复生成了

```shell
  ls ~/.ssh

  # 没有找到就直接重新生成一个
  ssh-keygen -C "your.email@example.com"
  
```

###### 把公钥上传到服务器，使用ssh-copy-id命令（如果是mac用户，需要安装一下该软件：brew install ssh-copy-id）

```shell
# 这个命令将会在你的服务器上，deployer1用户的home目录下.ssh/目录下创建一个authorized_keys文件，并把公钥存储到这个文件中
# 有些教程说是可以通过直接在服务器上创建这个文件，然后把本地的公钥复制黏贴到这个文件中，我试了一下，不成功，可能是哪里复制有问题，但用这个命令就可以
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 123456 deployer1@xxx.xxx.xxx.xxx
```

#### 使用ufw管理你的防火墙
此程序可以很方便的管理你的防火墙，其后台是iptable

```shell
# 安装ufw，如果没有安装 w
sudo apt-get install ufw
```
##### 启动ufw后，默认会关闭所有端口的连接，所以你需要通过一下的方式设置你需要访问的端口：

```shell
# 允许访问123456端口，协议包含：tcp和udp
sudo ufw allow 123456

# 如果服务器是用于网站用途，你需要打开80端口和443端口
sudo ufw allow 80
sudo ufw allow 443
```

更多的用法可以参考： http://wiki.ubuntu.org.cn/Ufw%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97
或者： sudo ufw -h

#### 挂载你的数据盘，官方有详细的教程（http://help.aliyun.com/knowledge_detail.htm?knowledgeId=5974154）

这里简单的罗列一下：

* 查看数据盘：fdisk -l

```shell
# 如果显示类似的以下信息，则说明你又数据盘
Disk /dev/xvdb xxx GB, xxxxx bytes
```

* 对数据盘进行分区

```shell
fdisk -S 56 /dev/xvdb
# 执行该命令后，你可以按照其提示一步一步进行设置

# 再用 fdisk -l 就可以查看刚分好区的云盘了
```

* 格式化新分区：

```shell
mkfs.ext3 /dev/xvdb1
```

* 添加分区信息

```shell
# 命令行中执行以下命令，其中：/mnt 为目标挂载点，你也可以换成自己想要的挂载点，比如： /home/deployer1/data
echo '/dev/xvdb1  /mnt ext3    defaults    0  0' >> /etc/fstab

# 查看是否写入成功
cat /etc/fstab
```

* 挂载新分区

```shell
mount -a

# 然后通过df -h 就可以查看到新添加的云盘了
```
ps：如果你再添加分区信息时，设置了自己的挂载点，你需要保证的自己的挂载点存在，如果不存在，就需要通过 mkdir新建一个目录

### 安装mysql，并把mysql的数据文件夹设置到云盘

#### 安装mysql

```shell
# 安装mysql
sudo apt-get install mysql-server

# ps 附上彻底卸载mysql的命令^_^
sudo apt-get autoremove --purge mysql-server mysql-server-5.0 mysql-common
```

#### 添加一个用户

```shell
grant all privileges on *.* to 'username'@'%' identified by 'password';

# 以上命令添加了一个用户到mysql，我们就可以用username 和 password登陆mysql
```

#### 设置外网可以访问

这个需要谨慎选择，有安全风险

```shell
# 打开mysql的端口，默认为3306端口
sudo ufw allow 3306

# mysql的配置文件（/etc/mysql/my.cnf）中，注释掉已下行
bind-address            = 127.0.0.1
```

#### 修改数据存放目录

##### 停止mysql

```shell
sudo service mysql stop
```

##### 复制/var/lib/mysql 这个文件夹到指定的目录，比如/home/data

```shell
mv /var/lib/mysql /home/data/
```

##### 修改mysql 配置文件：/etc/mysql/my.conf文件

找到： datadir = /var/lib/mysql

重新修改为： datadir = /home/data/mysql

##### 重启mysql：

```shell
sudo service mysql start
```

### 设置unicorn

#### 添加unicorn到Gemfile：

```shell
gem 'unicorn'
```

#### 在应用的config/目录下新建unicorn.rb配置文件:

```ruby
# Set the current app's path for later reference. Rails.root isn't available at
# this point, so we have to point up a directory.
app_path = File.expand_path(File.dirname(__FILE__) + '/..')

# The number of worker processes you have here should equal the number of CPU
# cores your server has.
worker_processes (ENV['RAILS_ENV'] == 'production' ? 4 : 1)

# You can listen on a port or a socket. Listening on a socket is good in a
# production environment, but listening on a port can be useful for local
# debugging purposes.
listen app_path + '/tmp/unicorn.sock', backlog: 64

# Time-out
timeout 300

# Set the working directory of this unicorn instance.
working_directory app_path

# Set the location of the unicorn pid file. This should match what we put in the
# unicorn init script later.
pid app_path + '/tmp/unicorn.pid'

# You should define your stderr and stdout here. If you don't, stderr defaults
# to /dev/null and you'll lose any error logging when in daemon mode.
stderr_path app_path + '/log/unicorn.log'
stdout_path app_path + '/log/unicorn.log'

# Load the app up before forking.
preload_app true

# Garbage collection settings.
GC.respond_to?(:copy_on_write_friendly=) &&
  GC.copy_on_write_friendly = true

# If using ActiveRecord, disconnect (from the database) before forking.
before_fork do |server, worker|
  defined?(ActiveRecord::Base) &&
    ActiveRecord::Base.connection.disconnect!
end

# After forking, restore your ActiveRecord connection.
after_fork do |server, worker|
  defined?(ActiveRecord::Base) &&
    ActiveRecord::Base.establish_connection
end
```

这个时候，可以通过已下命令测试一下rails 采用unicorn启动

```shell
# 在你自己的rails根目录下执行，就可以启动rails了
unicorn -c config/unicorn.rb
```

#### 设置unicorn启动脚本

```shell
#!/bin/sh

# File: /etc/init.d/unicorn

### BEGIN INIT INFO
# Provides:          unicorn
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the unicorn web server
# Description:       starts unicorn
### END INIT INFO

# Feel free to change any of the following variables for your app:

# ubuntu is the default user on Amazon's EC2 Ubuntu instances.
USER=deployer1

# Replace [PATH_TO_RAILS_ROOT_FOLDER] with your application's path. I prefer
# /srv/app-name to /var/www. The /srv folder is specified as the server's
# "service data" folder, where services are located. The /var directory,
# however, is dedicated to variable data that changes rapidly, such as logs.
# Reference https://help.ubuntu.com/community/LinuxFilesystemTreeOverview for
# more information.
APP_ROOT="/path/to/your/rails_root_path"

# Set the environment. This can be changed to staging or development for staging servers.
RAILS_ENV=production

# This should match the pid setting in $APP_ROOT/config/unicorn.rb.
PID=$APP_ROOT/tmp/unicorn.pid

# A simple description for service output.
DESC="Unicorn app - $RAILS_ENV"

# Unicorn can be run using `bundle exec unicorn` or `bin/unicorn`.
UNICORN="unicorn"

# Execute the unicorn executable as a daemon, with the appropriate configuration
# and in the appropriate environment.
UNICORN_OPTS="-c $APP_ROOT/config/unicorn.rb -E $RAILS_ENV -D"
CMD="RAILS_ENV=$RAILS_ENV $UNICORN $UNICORN_OPTS"

# Give your upgrade action a timeout of 60 seconds.
TIMEOUT=60
# end of custom options

# Store the action that we should take from the service command's first
# argument (e.g. start, stop, upgrade).
action="$1"

# Make sure the script exits if any variables are unset. This is short for
# set -o nounset.
set -u

# Set the location of the old pid. The old pid is the process that is getting replaced.
old_pid="$PID.oldbin"

# Make sure the APP_ROOT is actually a folder that exists. An error message from
# the cd command will be displayed if it fails.
cd $APP_ROOT || exit 1

# A function to send a signal to the current unicorn master process.
sig () {
  test -s "$PID" && kill -$1 `cat $PID`
}

# Send a signal to the old process.
oldsig () {
  test -s $old_pid && kill -$1 `cat $old_pid`
}

# A switch for handling the possible actions to take on the unicorn process.
case $action in
  # Start the process by testing if it's there (sig 0), failing if it is,
  # otherwise running the command as specified above.
  start)
    sig 0 && echo >&2 "$DESC is already running" && exit 0
    su - $USER -c "$CMD"
    ;;

  # Graceful shutdown. Send QUIT signal to the process. Requests will be
  # completed before the processes are terminated.
  stop)
    sig QUIT && echo "Stopping $DESC" exit 0
    echo >&2 "Not running"
    ;;

  # Quick shutdown - kills all workers immediately.
  force-stop)
    sig TERM && echo "Force-stopping $DESC" && exit 0
    echo >&2 "Not running"
    ;;

  # Graceful shutdown and then start.
  restart)
    sig QUIT && echo "Restarting $DESC" && sleep 2 \
      && su - $USER -c "$CMD" && exit 0
    echo >&2 "Couldn't restart."
    ;;

  # Reloads config file (unicorn.rb) and gracefully restarts all workers. This
  # command won't pick up application code changes if you have `preload_app
  # true` in your unicorn.rb config file.
  reload)
    sig HUP && echo "Reloading configuration for $DESC" && exit 0
    echo >&2 "Couldn't reload configuration."
    ;;

  # Re-execute the running binary, then gracefully shutdown old process. This
  # command allows you to have zero-downtime deployments. The application may
  # spin for a minute, but at least the user doesn't get a 500 error page or
  # the like. Unicorn interprets the USR2 signal as a request to start a new
  # master process and phase out the old worker processes. If the upgrade fails
  # for some reason, a new process is started.
  upgrade)
    if sig USR2 && echo "Upgrading $DESC" && sleep 10 \
      && sig 0 && oldsig QUIT
    then
      n=$TIMEOUT
      while test -s $old_pid && test $n -ge 0
      do
        printf '.' && sleep 1 && n=$(( $n - 1 ))
      done
      echo

      if test $n -lt 0 && test -s $old_pid
      then
        echo >&2 "$old_pid still exists after $TIMEOUT seconds"
        exit 1
      fi
      exit 0
    fi
    echo >&2 "Couldn't upgrade, starting 'su - $USER -c \"$CMD\"' instead"
    su - $USER -c "$CMD"
    ;;

  # A basic status checker. Just checks if the master process is responding to
  # the `kill` command.
  status)
    sig 0 && echo >&2 "$DESC is running." && exit 0
    echo >&2 "$DESC is not running."
    ;;

  # Reopen all logs owned by the master and all workers.
  reopen-logs)
    sig USR1
    ;;

  # Any other action gets the usage message.
  *)
    # Usage
    echo >&2 "Usage: $0 <start|stop|restart|reload|upgrade|force-stop|reopen-logs>"
    exit 1
    ;;
esac
```

### 配置nginx

#### 安装nginx

```shell
sudo apt-get install
```

#### 配置nginx脚本，新建文件/etc/nginx/sites-available/sitename文件，添加如下的配置项

```shell
upstream app {
        # 这里的路径设置一定要和unicorn中配置的sock位置保持一致，不然nginx和unicorn将无法通信
        server unix:/path/to/your/rails_app_path/tmp/unicorn.sock fail_timeout=0;
}

server {
        listen   80; ## listen for ipv4; this line is default and implied
        server_name localhost;

        root /path/to/your/rails_app_path/public;

        try_files $uri/index.html $uri @app;

        # serve静态资源
        location ^~/assets/ {
                gzip_static on;
                expires max;
                add_header Cache-Control public;
        }

        location @app {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;
                proxy_pass http://app;
        }

        error_page 500 502 503 504 /500.html;
        client_max_body_size 4G;
        keepalive_timeout 10;
}
```

#### 在/etc/nginx/sites-enabled/文件夹下添加指向/etc/nginx/sites-available/sitename文件的软连接

```shell
sudo ln -s ../sites-available/sitename sitename
```


### 参考链接

http://wiki.ubuntu.org.cn/Ufw%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97
http://help.aliyun.com/knowledge_detail.htm?knowledgeId=5974154
http://www.gotealeaf.com/blog/setting-up-your-production-server-with-nginx-and-unicorn
http://vladigleba.com/blog/2014/03/27/deploying-rails-apps-part-4-configuring-nginx/




