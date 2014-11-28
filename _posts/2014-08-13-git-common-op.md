---
layout: post
title: "Git常用操作简记"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 基本操作
```shell
git clone git链接 #克隆remote代码
git clone -b branch名字 git链接 #指定remote的分支克隆
git branch #查看本地分支
git branch -d [name] #删除分支, -d选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用-D选项
git push origin --delete branch_name #删除remote分支
git branch -r #查看remote分支
git branch b-name #创建分支
git checkout b-name #切换到另外的分支
git checkout -b b-name #切换到remote的分支
git push origin b-name #本地branch push到remote
git config -l #查看本地git配置信息
```

### 进阶操作
```shell
git tag -a v1.0.0 -m "后台第一个版本！" #创建tag
git tag #列出本地的tag
git push origin --tags #本地所有tag一起push到remote
git push origin v1.0.0 #push本地特定的tag到remote

git log --pretty=format:'%h : %s' --topo-order --graph #log显示

#合并多个commit
git rebase -i HEAD~4 #对最近的4次进行合并
#rebase操作参考： http://blog.chinaunix.net/uid-27714502-id-3436706.html

#stash用法：
git stash #备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
git stash pop #从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。
git stash list #显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。
git stash clear #清空Git栈。此时使用gitg等图形化工具会发现，原来stash的哪些节点都消失了。
git stash apply stash@{1} #就可以将你指定版本号为stash@{1}的工作取出来

git reset --hard commit_id #回到某个版本
git reset --hard HEAD~3 #会将最新的3次提交全部重置，就像没有提交过一样

git remote set-url origin url #切换git url

#分支1 merge 到 分支2
git checkout branch2
git fetch origin
git merge origin/branch1 #merge完后需要可能需要手动处理conflict
git push origin branch2
```

### github托管代码（linux系统）
- 注册github；
- 安装git：

```shell
sudo apt-get install git
```

- 生成github公钥和私钥：

```shell
ssh-keygen -C "github帐号" -f ~/.ssh/github
```

- 然后把~/.ssh/github目录下的github.pub文件中的公钥拷贝到github网站个人设置中的SSH keys中新添加的ssh中
- 测试是否设置正确：

```shell
ssh -T git@github.com
```

> Hi XXX! You've successfully authenticated, but GitHub does not provide shell access. #显示这个表示设置成功

- 完成配置，可以从githubclone代码了
- 本地创建git库，然后push到github

```shell
# 先进入需要push的目录
git init
git commit -m "first commit"
git remote add origin git@github.com:xxx/xxx.git
git push -u origin master
```
