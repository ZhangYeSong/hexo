---
title: Could not restore untracked files from stash —— git stash -a 之后的事故处理
date: 2018-10-08 21:05:09
categories:
 - git
tags:
 - git
---
> 今天在使用stash的时候鬼使神差地打了个-a，然后就把我给吓到了。。。

在git commit的后面加一个-a，会把untracked文件也全部add进去。今天stash的时候也突然鬼使神差地加了个-a，看起来似乎没有问题。然后在stash apply的时候悲剧了：

```
.../xxx/temp/xxx.jar already exists, no checkout
.../xxx/temp/xxx.jar already exists, no checkout
.../xxx/temp/xxx.jar already exists, no checkout
.../xxx/xxx.war already exists, no checkout
Could not restore untracked files from stash
```
<!-- more -->
随便百度了下貌似没找到啥答案，都是说不要加-a，加-u。先来看下为啥会出现这样的问题吧。

在终端输入man git stash看一下，有这么一行：    

       git stash [save [-p|--patch] [-k|--[no-]keep-index] [-q|--quiet]
                    [-u|--include-untracked] [-a|--all] [<message>]]
可以看到-u是包含所有untracked的文件，而-a是包含所有文件。这俩有啥区别呢？-u不会把.gitignore内忽略的文件添加到stash中，而-a是不会管忽略不忽略的。这就非常坑了，因为我们的项目各种build目录文件太多太多了，全给添加进去导致我们apply的时候还原不回来了-_-

咋办呢，还好翻到了一篇2012年的博客救了我https://blog.tfnico.com/2012/09/git-stash-blooper-could-not-restore.html

里面是这样说的，先喝口水冷静下来：*A stash is basically a commit.*

啥意思？其实stash的记录也是基于commit来做的，所以先想办法查到stash记录的commit id。

使用git log --graph --all --decorate --oneline命令来查这个id。说实话在此之前我还真不知道git log可以有这么多指令，于是去查了下：--graph意思是以图形的模式显示分支， --all意思是显示所有的commit记录，--decorate参数用来显示一些相关的信息，如HEAD、分支名、tag名等，--oneline就是一个commit只显示一行。

```
- dd7b26b (master) Extract method
- b0d0487 Rename one of the assertScreenForEvent methods
- ddbb532 Use strong typing for Events
  | *-.   **0374bd1** (refs/stash) WIP on master: d7fe9ab new class Event
  | |\ \  
  | | | * 2a958c1 untracked files on master: d7fe9ab new class Event
  | | * 51510de index on master: d7fe9ab new class Event
  | |/  
  | * d7fe9ab (HEAD) new class Event
  | * c59f0f3 Extract method
  | * 74a987f Rename one of the assertScreenForEvent methods
  | * 75e98b7 Use strong typing for Events
  |/  
  | * 40b5b29 (origin/master, origin/HEAD) added language-chooser
```

0374bd1就是我们要找的那笔出问题的stash的commit id。

接下来输入git checkout 0374bd1， 切换到该提交的分支。

然后git reset HEAD~1，把commit reset掉，重新使用git stash -u保存修改到stash中。

然后git checkout master切换到之前的分支，重新git stash apply或者pop还原修改就ok了。

最后再分享下，其实git reset的是用了--hard也不用担心，git reflog还是能找到reset之前的记录，当然太过久远的可能就没了。

好了，大功告成。这件小事让我明白git上的stash和commit记录几乎是很难丢失的。遇到问题不要担心自己的代码没了，先喝杯水压压惊，肯定有办法的！
