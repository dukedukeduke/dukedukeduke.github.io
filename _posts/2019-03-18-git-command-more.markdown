---
layout: post
title:  "git 常用命令-more"
date:   2019-03-18 16:27:02 +0800
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

### git stash
git stash命令用于当前进度保存起来，git stash pop为切换为最近一个stash保存的状态， 会将切换前的代码和切换后的代码进行合并，如果代码有冲突就需要进行冲突解决。
比如正在开发某个功能，正在编写途中，但是突然有临时bug需要修改，这是需要将已经开发的部分代码git stash临时保存起来
#### 1. 我修改了一个文件index.js:

```
stm-macmini$ git status
On branch ruled
Your branch is up-to-date with 'origin/ruled'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.DS_Store
	.idea/
```

#### 2. git stash:

```
stm-macmini$ git stash
Saved working directory and index state WIP on ruled: 06037d9 add download hosting files function, add  fetching food0001 resources info function
stm-macmini$ git status
On branch ruled
Your branch is up-to-date with 'origin/ruled'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.DS_Store
	.idea/
```

发现提示已经暂存了之前的改动， 同时会生成干净的代码（改动前。
#### 3. 进行bug代码的修复：

```
stm-macmini$ git status
On branch ruled
Your branch is up-to-date with 'origin/ruled'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.DS_Store
	.idea/

no changes added to commit (use "git add" and/or "git commit -a")
stm-macmini$ git diff index.js
diff --git a/index.js b/index.js
index 6a66411..1c84095 100755
--- a/index.js
+++ b/index.js
@@ -228,6 +228,7 @@ function checkDate() {


 (async function () {
+    console.log("123");
```

#### 4. bug代码 git add, git commit ,git push:

```
stm-macmini$ git add index.js
stm-macmini$ git commit -m "test"
[ruled 806e241] test
 1 file changed, 1 insertion(+)
stm-macmini$ git push origin ruled
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 309 bytes | 309.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
```

#### 5. git stash pop：
如果bug修复代码和开发了部分的功能代码有代码重叠，会发现冲突提示，如下：

```
stm-macmini-090:tool_github_data_exporter-node yangchun$ git stash pop
Auto-merging index.js
CONFLICT (content): Merge conflict in index.js
```

#### 6. 进行冲突修复：
VCS -> Git -> Resolve Conflicts -> MERGE,中间代码为最终的合并结果，点击apply即可生效。

如果修改完bug代码不进行commit和push,步骤如下：
#### 4. git add, git commit -m "提交bug修复代码"

```
stm-macmini-090:test yangchun$ git commit -m "test"
[master e659b1b] test
 1 file changed, 1 insertion(+), 1 deletion(-)
```

#### 5. git stash pop

```
stm-macmini-090:test yangchun$ git stash pop
Auto-merging help.md
CONFLICT (content): Merge conflict in help.md
```

#### 处理冲突
同上

#### 注意
处理完冲突之后，已经执行了git add命令。

### git add
git add命令主要用于把我们要提交的文件的信息添加到索引库中。当我们使用git commit时，git将依据索引库中的内容来进行文件的提交.
通常是通过git add <path>的形式把我们<path>添加到索引库中，<path>可以是文件也可以是目录

### git commit
git commit 主要是将暂存区里的改动给提交到本地的版本库。每次使用git commit 命令我们都会在本地版本库生成一个40位的哈希值，这个哈希值也叫commit-id，commit-id在版本回退的时候是非常有用的，它相当于一个快照,可以在未来的任何时候通过与git reset的组合命令回到这里.
####  git commit -a -m “massage”:
-a参数可以将所有已跟踪文件中的执行修改或删除操作的文件都提交到本地仓库，即使它们没有经过git add添加到暂存区，注意，新加的文件（即没有被git系统管理的文件）是不能被提交到本地仓库的。建议一般不要使用-a参数，正常的提交还是使用git add先将要改动的文件添加到暂存区，再用git commit 提交到本地版本库
 
 ### 在自己修改代码同时，远端仓库有代码更新
 先add和commit自己的改动代码到本地仓库， 然后git pull，如果有冲突，就按上述方式解决冲突， 最后push 到远端仓库
