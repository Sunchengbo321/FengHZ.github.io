---
layout: post
title: Git 学习笔记
date: 2018-04-16 13:49:00
categories: 实用工具
tags: Git
mathjax: true
---
* content
{:toc}

# Git学习内容:多分支与回退版本

## 学习目的

1.弄清楚如何创建分支，合并分支，以及分支开发到一半回退回主枝

2.弄清楚如何把文件回退到指定的版本

**本学习所有内容都来自于网站[Git Book 中文版](https://git-scm.com/book/zh/v2)中*chap 2-3 ,7*的内容**




## 多分支创建

### 问题背景

在写一个比较长的程序的时候，我们常常会遇到的一个实用场景是我想在程序上加一个功能或者是测试一种情况。进行这种操作需要在已有的工作文件加很多额外的文件与代码，但是这种操作可能就是一个test，我们希望**做完操作后回到原来的样子，或者做完操作后将新文件与原来的文件合并**，或者做完操作后把这些内容暂存，然后继续在原程序上开发。

这些问题都可以用分支来进行解决。

所谓的分支，顾名思义就是在主干上另外开辟一个存储路径来存储新的版本，类似于我们将主干文件复制一遍然后在复制的上面进行开发。复制操作非常浪费时间也不优雅，我们可以用Git系统来创建，合并，删除分支。

### 分支区分

在进入git系统管理的文件夹后，默认分支名字一般叫`master`，代表我们的主要开发分支。在当前的主支上开辟一个新分支，名字为`new-branch`的命令为：

```shell
git branch new-branch
```

切换到该新分支的命令为

```shell
git checkout new-branch
```

或者我们可以用一个命令创建并到新分支处

```shell
git checkout -b new-branch
```

查看当前文件夹下有多少分支，当前处在哪一个分支下的命令为

```shell
git log --oneline --decorate
```

它的一般输出为

```shell
e360362 (HEAD -> testing) made a change
a4bfeef (master) 6666
3448b0f Try "commit"
a9e378a initial project version
```

前面为某次提交对应的版本识别字符串，为`SHA-1`编码的。后面为每一次提交的`message`，而第一行`Head->branchname`指向的branch即为当前所处的branch

或者直接用命令

```shell
git branch
```

打\*的为当前分支。

### 退回新分支

如果我们对新分支下的文件进行了操作（即此时先在命令行内把分支进行了切换，然后对分支内的文件进行了操作）后，我们想要回到原分支，此时有2种可能，一种可能是我现在原分支里面把更改直接使用`git commit -a -m "message"`命令先提交好，然后采用

```shell
git checkout master
```

来退回到原分支。此时commit仅是提交到了当前分支下，对原分支没有影响。但是如果我们在新分支下写了一堆garbage，然后不想提交，想之后处理的话，我们可以采用

```shell
git stash save
```

命令来把当前的garbage暂存一下，并不提交，而当前的工作目录状态回到当前分支上一个提交的状态。

我们可以用

```shell
git stash list
git stash apply
or
git stash apply --index
```

来回到我们暂存的状态。但是无论分支上怎么开发，都不会影响到主支的。

或者说我们可以用

```shell
git stash branch <branchname>
```

来新建一个仅基于garbage状态的新分支进行开发，这用于garbage的内容与当前开发内容有矛盾的时候。

### 枝干合并

如果我们在分支上干的很满意，就可以直接和主支合并了，此时可以直接用

```shell
git checkout master
git merge new-branch
```

来合并分支，当然如果合并遇到冲突的地方，文件会标出来并出现一些特殊区段标出让你选择一种，或自行更改作为合并。更改后，对冲突文件使用`git add`命令来把冲突文件加入标记为冲突解决。

或者可以用

```shell
git mergetool
```

来进行可视化的合并。

### 无用枝干清除

我们可以采用

```shell
git branch --merged
```

来查看哪些分支已经被合并了，加入它的输出为

```shell
  iss53
* master
```

这意味着所有分支已经合并到了标*的分支，这个例子为iss53分支合并到了master分支，我们应该使用

```shell
git branch -d iss53
```

来删除该分支。而同时

```shell
git branch --no-merged
```

表示的是当前没有合并的分支，如果要删除这些分支需要用

```shell
git branch -D branch-name
```

来完成。

## 版本控制

这个版本控制的具体问题是，我在当前的branch下进行了一些工作，进行了一些提交，但是我发现某一次提交犯了一个很大很大很大的错误，我需要回退到以前的某个提交版本去避免这个错误，重新开始的问题。

假设在我做了一次错误提交之后我的电脑存储了若干个提交版本，采用命令

```shell
git log --pretty=oneline
```

来查看提交历史，假设提交历史为:

```shell
ab1afef80fac8e34258ff41fc1b867c702daa24b modified repo a bit
484a59275031909e19aadb7c92262719cfcdf19a added repo.rb
1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
cac0cab538b970a37ea1e769cbbde608743bc96d second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```

我想要回到版本编码为la410的版本，我有2个办法：

### 强行重置回去

强行回退到之前的某个版本：

```shell
git reset --hard 1a410efbd13591db07496601ebc7a059dd55cfe9
```

此时master分支这个版本之后的所有提交都会丢失，当然不是不可逆的丢失，是可以找回的。

或者我们可以创建一个指向这个目标版本的新分支来恢复它而不影响主分支：

```shell
git branch new-branch 1a410efbd13591db07496601ebc7a059dd55cfe9
```

## 其它一些有用的小tips

因为git对历史的超高保存性，这一方面确保了我们删东西总是可以恢复的，但是另一方面删东西也确实不容易。假设有一个人在一个历史版本中提交了一个超大的文件，而它在最新的提交中已经被

```
git rm <filename>
```

给删掉了，但是因为它一直保存在历史版本中，这就导致该文件一直会被下载，为了保存历史文件的完整性。此时需要在网页[git 文件恢复与删除](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E7%BB%B4%E6%8A%A4%E4%B8%8E%E6%95%B0%E6%8D%AE%E6%81%A2%E5%A4%8D)中采用`git -filter-branch`命令来删除。因此一个很重要的是在第一次提交之前检查`.gitignore`的内容，从而尽量不出现差错，或者在出现问题的第二次提交的时候就把这个东西给remove掉就可以了。

