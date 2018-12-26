---
title: Alfred+Workflow让工作效率升级
date: 2018-12-26 23:18:20
categories:
 - 随笔
tags:
 - 环境配置
---
> Alfred是Mac系统上的一款效率软件，它的免费功能就很强大了，最近终于狠下心来买了Mega License，使用下来感觉花的还是值得的。

### Alfred
对于类Unix系统（Mac OSX和Linux都属于），shell命令就是最好的效率工具，熟练使用shell命令可以极大地提升工作效率。而Mac系统相比于Linux系统还有一个独占的效率神器——Alfred。
[Alfred官方地址](https://www.alfredapp.com/)，免费版的Alfred我基本上是当软件启动器使用的，即使如此也给我使用Mac带来很大的方便，而且实测搜索效果明显好于Mac自带的Spotlight(现在好像改名聚焦搜索了)。在Winodws上我使用[wox](http://www.wox.one/)，在Linux上使用[Albert](https://albertlauncher.github.io/docs/installing/)来代替Alfred。不过使用了付费版的Alfred之后以后可能不想再回到linux和windows开发环境了- -
另外我设置的Alfred启动快捷键是连续按两下command键，感觉这样非常的方便。

### License
[购买License地址](https://www.alfredapp.com/powerpack/buy/)有两个License如下：
![购买地址](https://upload-images.jianshu.io/upload_images/5586297-4a68f9b1c8d473d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
单位上英镑，两个都是针对单人的License，左边的只针对当前的V3版本，后续升级大版本还需要继续购买不过有折扣；右边的是终身免费升级版本。我买的是右边的Mega License，折合人民币大概357元。我知道有很多人使用破解版，但是有经济实力的还是支持一下开发者吧。
<!-- more -->

### File Search
这是一个非常强大的功能。在Windows上有everything这款软件，搜索文件非常的快，但是我们的Alfred搜索起来也是一样的超快，并且支持很多其他功能。语言上可能描述不太好，直接上截图吧：
![普通地搜索文件](https://upload-images.jianshu.io/upload_images/5586297-adee1c8db5163fdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![按内容搜索](https://upload-images.jianshu.io/upload_images/5586297-db8bc3deef83a4d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![搜索文件夹后按前后键进入上一级下一级，需要设置](https://upload-images.jianshu.io/upload_images/5586297-71cdb4440038e19b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![找出文件后按tab键出来actions，需要设置](https://upload-images.jianshu.io/upload_images/5586297-fa23286cf143d5c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
怎么样，是不是超级灵活超级强大？讲真Mac上的Finder真的是太烂了，但是我们有了Alfred+shell命令，完全不用再怀念Windows的资源管理器了！

### Web Search
这里分享两个我自定义的Web Search网址：
- 淘宝：http://s.taobao.com/search?oe=utf-8&f=8&q={query}
- 百度：http://www.baidu.com/s?wd={query}
再加上系统自带的那些基本够用了，其他网站可以上网查一下。
另外记得把Web bookmarks里面的选项打开，这样打开网站非常迅速。

### Dictionary
workflow里面有不少翻译相关的flow，但是我觉得自带的词典app就很不错了，只需要打spell，自动识别语言。另外推荐一个npm里面的翻译工具[cli-dict](https://www.npmjs.com/package/cli-dict/v/0.0.54)也是自动识别中英文，非常地方便。

### Clipboard
非常强大的查询粘贴板历史的工具，图片文件复制历史也能查，建议把text历史保留时间搞到三个月- -
另外shell命令history我也很常用，非常轻易的能搜索到最近使用的命令。

### Snippets
![Snippets](https://upload-images.jianshu.io/upload_images/5586297-0793b9d3197ae414.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对我来说就是个大段文本生成器，但是实际能做的事情非常多[官网介绍](https://www.alfredapp.com/help/features/snippets/)，用好了对效率的提升毋庸置疑。这个好像用某些输入法也能实现？但是我不喜欢安装各种广告和乱七八糟后台占用的第三方输入法，因此这个功能还是很有用的。以后可以收集代码片段来做snippets。

### Workflow
最后才讲到workflow哈哈～目前我还不会自己做workflow，如果有大量重复的日常工作的话自制workflow还是很有必要的。比较尴尬的是官方没有做专门用来发布workflow的网站，要找别人做好的workflow主要以下三种渠道：
-  [alfred官方论坛](https://www.alfredforum.com/)
- [官方指定的发布网站Packal](http://www.packal.org/) 但实际上还是个人做的，网站bug挺多- -
- [github自己搜索](https://github.com)
来介绍下我现在用的一些workflow吧：
- [Packal Workflow Search](http://www.packal.org/workflow/packal-workflow-search)
用来在Packal查找workflow的workflow- -
- [fixum](https://github.com/deanishe/alfred-fixum)
有些workflow可能版本过旧运行不了，那么就需要这个workflow来fix以下就OK啦
- [Dash](https://kapeli.com/dash)
Dash是Mac上非常强大的各种sdk文档查询软件，可以说是程序员的神器了，配合上Alfred更是威力无边。而且Dash和sublime一样都是可以永久免费试用的软件。这个直接在Dash的设置中关联workflow即可。
- [Github](http://www.packal.org/workflow/github)
非常强大的关联Github的workflow，不止是搜索噢～
- [http-status-codes](http://www.packal.org/workflow/http-status-codes)
Dash上还真查不到http code，每次都要查很麻烦，有了这个就方便太多啦！
- [mvnrepository](http://www.packal.org/workflow/mvnrepository)
有关maven的workflow很多，不过这个应该上最强大的，可以选择复制出来的是maven格式的还是gradle格式的，只能说非常非常的强！虽说jetbrain家的IDE也能搜索，但是那个搜索实测远不如这个体验好。
- [stackoverflow-search](http://www.packal.org/workflow/stackoverflow-search)
程序员的救命网站，使用workflow让救援来的更快一点！！！
- [NpmSearch](http://www.packal.org/workflow/npmsearch)
搜索npm的workflow

暂时安装的就这么多了，有些命令行能搞定的就没安装了比如kill process，top之类的，用太多把shell命令技能给退化了就不好了。个人认为shell命令和alfred互补使用是最好的。
有了workflow写起博客来都效率升级啦，还等啥赶快购买吧！
