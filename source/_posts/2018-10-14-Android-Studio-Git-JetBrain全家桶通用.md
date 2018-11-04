---
title: Android Studio + Git(JetBrain全家桶通用)
date: 2018-10-14 21:04:47
categories:
 - git
tags:
 - git
 - Android Studio
 - Android
 - 环境配置
---
> 观察了下身边同事操作git基本上都是用git命令行或者tortoise git。但是在windows下使用命令行得先启动bash，有点麻烦。因此使用Android Studio自带的图形化版本控制工具还是能带来一些便捷性的。当然其他JetBrain系列的IDE如IntelliJ Idea、WebStorm等都通用。

### 0.开启git版本控制
首先先在setting里设置git可执行文件

![设置git可执行文件](https://upload-images.jianshu.io/upload_images/5586297-46c58ca16d4005d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!-- more -->

点击test可以测试下，我这是Mac下的路径，windows下的就看自己git安装在哪里了。
同时AS也深度支持GitHub，可以顺便配置下自己GitHub的帐号。

对于本身就是使用git clone下来的项目，git是默认打开的。使用Android Studio拉取项目可以在选取项目界面中选择Check out Project from Version Control。
![从远程仓库拉取项目](https://upload-images.jianshu.io/upload_images/5586297-de559cbb712a90ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后选择git或者GitHub，非常方便。

对于还没有开启git控制的项目，可以点击VCS -> Enable Version Control Integration然后选择git

![开启git版本控制](https://upload-images.jianshu.io/upload_images/5586297-21b4229956ce6e2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![开启git版本控制](https://upload-images.jianshu.io/upload_images/5586297-440e36551c396d43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样我们就可以开始选择用Android Studio来进行git操作了。

### 1.commit／pull／push等基本操作
这些基本操作在任意文件下右键或者点击File下选择git即可：
![git基本操作](https://upload-images.jianshu.io/upload_images/5586297-5400893c8c972a71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以说毫无难度。Anotate下面单独说。Show Current Revision是显示当前文件最近的一次修改记录；Compare几个操作数对比差异；Show History和git log的作用相同，显示文件的所有历史修改记录。Revert是把当前修改未commit的内容还原，非常的方便。如果有要pull、push的commit，也会显示在这里面，或者直接点击上方工具栏里的图标更方便。

### 2.Annotation的妙用
点击上面的Annotation，或者在文件的左侧行数上右键点击Annotation，会显示当前文件每一行的最近提交记录，包含时间和提交人以及commit信息。

![Annotation](https://upload-images.jianshu.io/upload_images/5586297-f8005869e4a532c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
有*号的表示是未提交代码。当然我这里是一个个人项目，只有我自己的提交记录。
这个有什么用呢？如果是多人开发项目，意义非常大，可以让你以最快的速度甩锅（开个玩笑- -）。在开发人数非常多的时候，我们不可避免的要经常读别人的代码。当我们看不懂某行代码时就可以第一时间通过Annotation找到提交人，然后问他。如果想看是谁动了自己模块的代码，也是一样的简单。每当测试发出一个crash记录时，立马定位到代码看下Annotation，是不是自己的锅。。。。。。

### 3.Gerrit push不了的问题解决
有的同学会发现，用studio内置的图形化工具gerrit的仓库push不上去。仔细想一下，提交gerrit仓库的时候我们是这样写的：
```
git push origin HEAD:refs/for/xxxx
```
也就是说提交的远程分支上要加一个refs/for，这个也很简单，如下，我们在push的时候把这段加上就可以了，一般改一次就行了。
![gerrit push加上refs/for](https://upload-images.jianshu.io/upload_images/5586297-1a8e1760587d5fa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4.Changelist的使用
有时候我们需要在本地修改一些代码便于测试调试，比如数据库是加密的，我们本地需要改成不加密的然后才能用工具调试sqlite数据库。但是这个代码又不能提交到远程仓库上。这样我们每次提交的时候都需要小心翼翼的把这个文件勾掉，或者还原掉，很麻烦。这个时候就轮到Changelist出场了：
![Changelist](https://upload-images.jianshu.io/upload_images/5586297-2cf6b588acb0b9e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![创建Changelist](https://upload-images.jianshu.io/upload_images/5586297-a28d1094a32d361a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击下方的Version Control，在Local Changes里选择我们不想提交的文件右键选择Move to Another Changelist, 然后新建一个Changelist，注意左下的Set active不要勾选。没有勾选active的Changelist默认是不会在commit的时候提交的。
然后在我们commit的时候就不会出现这些文件，当然commit的时候也是可以选择其他Changelist来提交这些文件的。

### 5.结语
JetBrain家做的IDE真的是无比强大，肯定还有些其他功能我没有发现的，大家可以回复告诉我。当然，用git命令行肯定也是必不可少的，目前我处理复杂的git问题还是会使用命令行，普通的pull／push使用IDE自带的图形化工具。最后吐槽一下Mac OS的截图是真的麻烦，截完还要自己去手动命名，真的是不如windows下截图方便。
