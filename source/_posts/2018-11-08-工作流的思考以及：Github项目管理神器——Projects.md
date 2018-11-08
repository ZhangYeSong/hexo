---
title: 工作流的思考以及：Github项目管理神器——Projects
date: 2018-11-08 21:06:36
categories: 
 - 项目管理
tags: 
 - 项目管理
 - Github
---
> 我突然明白，提前完成工作的成就感对我是砒霜。

最近手上活有点紧，被迫想寻找点改变来优化自己的工作流程，提升工作效率。第一个想到的就是计划，有计划地工作对效率的提升是巨大的。在以往很长的一段时间里，我写代码都是狼奔豕突式的疯狂突击。每次写代码就像是一次次冲锋，试图快速干完需求。

先不谈这样写代码or工作的弊端，溯根求源，来分析下我为啥总喜欢这样的明显不好的方式写代码：表面上是想早点写完然后可以休息或者干其他事情，但我真诚地审视了下自己根本原因不是这样。根本原因是我对工作这件事上没有足够的信心，我希望快速把事情干完证明自己可以，害怕事情丢到明天完不成。

冲锋式地工作在我提前完成工作的时候会让自己非常有成就感，但现在我才明白这种无谓的成就感是砒霜。它会让我只过分关注结果而忽略过程和质量，让我自我感觉良好。而如果一时攻克不下，那就更糟糕了，往往我会选择死磕很久。写代码这件事和其他工作相比，很多时候死磕是不可避免的。即使你已经冷静地debug，冷静地分析日志，向同事、同行、google、stackoverflow求助，还是会有很多时候找不到答案而疯狂。死磕不可避免，但是人的精力是有限的。死磕问题是最耗费精力的事，而冲锋式地工作往往会逼迫我自己去死磕，从而让自己身体和心理上都遭受沉重打击。
<!-- more -->

有了想法之后，我想找一个类似JIRA这样的项目管理工具来管理我的开发进度。首先就想到了github上的Marketplace，居然真的有Jira Software + GitHub这个工具。但是JIRA对我来说太重了，毕竟我想管理的是我自己一个人的开发进度，于是想看看还有没有别的选择。原来Github仓库里自带的就有项目管理工具，也就是Projects，真的是孤陋寡闻了。

![点击Projects](https://upload-images.jianshu.io/upload_images/5586297-5e8185aef16d27d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击Projects可以进入到项目管理界面，会显示当前在进行的project列表和进度。

![new project](https://upload-images.jianshu.io/upload_images/5586297-6fe4874f0da4c4f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击New Project可以创建项目，填写标题，描述，还可以选择模版。这里我选择的Basic kanban模版（看到这里我突然想到为啥这里会有拼音看板？查了下才知道这是英文引进的日语单词，也算变相地从中文引入了）。

![project-board.png](https://upload-images.jianshu.io/upload_images/5586297-55751d8d8cee34bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里就是生成的看板界面了，默认分成三个column，Todo、In progress、Done，可以把任务卡片随意拖动，也可以自定义添加新的column。

![edit-note.png](https://upload-images.jianshu.io/upload_images/5586297-76f4084516ddfab0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到每个卡片里面还可以有check list，按照格式去写就行了，其他语法基本和github的markdown语法差不多。

![demo-projects.png](https://upload-images.jianshu.io/upload_images/5586297-7efeb4d066ab2092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后，自己给自己手头的工作搞了一个任务看板，感觉真的非常不错。在规划任务的时候其实就是规划代码的过程，知道自己还有多少事要做，做了多少事，对于工作真的太有帮助了。真正做到心里有数，就不怕deadline，再也不要像个无头苍蝇去无计划地写代码啦！
