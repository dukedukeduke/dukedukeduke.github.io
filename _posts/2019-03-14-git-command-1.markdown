---
layout: post
title:  "git 常用命令"
date:   2019-03-14 20:27:02 +0800
comments: true
tags:
- git
---

### 参考
https://blog.csdn.net/u014540717/article/details/54314126

http://www.ruanyifeng.com/blog/2014/06/git_remote.html

https://www.w3cschool.cn/git/git-branch.html

### git clone
从远程主机克隆一个版本库

```
$ git clone <版本库的网址> <本地目录名>
```

<本地目录名>可省略

```
$ git clone https://github.com/jquery/jquery.git
```

克隆版本库的时候，所使用的远程主机自动被Git命名为origin

### git checkout
查看本地分支

```
stm-macmini$ git branch
  master
* ruled
```

查看远端分支

```
stm-macmini$ git branch -a
  master
* ruled
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/ruled
```

切换分支

```
stm-macmini$ git checkout ruled
Switched to branch 'ruled'
Your branch is up-to-date with 'origin/ruled'.

...

stm-macmini$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
```

### git remote
git remote命令列出所有远程主机

```
stm-macmini$ git remote
origin
```

-v选项，可以参看远程主机的网址

```
stm-macmini$ git remote -v
origin	http://github.com/tool_github_data.git (fetch)
origin	http://github.com/tool_github_data.git (push)
```

只有一台远程主机，叫做origin.

### git pull
git pull命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并
取回origin主机的next分支，与本地的master分支合并:

```
git pull origin next:master
```

省略冒号后面的本地分支则pull下来与当前分支合并, 而：

```
git pull
```

表示当前分支自动与唯一一个追踪分支进行合并。
如果远程主机删除了某个分支，默认情况下，git pull 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致git pull不知不觉删除了本地分支，加上参数 -p 就会在本地删除远程已经删除的分支。

```
git pull -p
```

### git push
用于将本地分支的更新，推送到远程主机.

```
git push <远程主机名> <本地分支名>:<远程分支名>
```

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

```
git push origin master
```

表示将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支,如：

```
git push origin :master
```

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

```
git push origin
```

表示将当前分支推送到origin主机的对应分支.

如果当前分支与多个主机存在追踪关系，则可以使用-u选项指定一个默认主机:

```
git push -u origin master
```

表示将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。

不带任何参数的git push，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。如果要修改这个设置，可以采用git config命令。

```
git config --global push.default matching
# 或者
git config --global push.default simple
```
