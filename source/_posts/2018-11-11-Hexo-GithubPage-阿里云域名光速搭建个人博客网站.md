---
title: Hexo+GithubPage+阿里云域名光速搭建个人博客网站
date: 2018-11-11 17:29:06
categories:
 - Node.js
tags:
 - Node.js
---
> 一直以为自己搭建博客会非常麻烦，结果试了下很有名的hexo框架，真的是光速建站。
### 为何要建个人博客网站
已经有简书之类的博客网站了，为啥还要自己建一个？这个原因有很多，比如想学学建站知识，想使用个人域名等等。对于我来说个人网站比较自由吧，在不违法的情况下可以随便写自己想说的。对于程序员来说，拥有一个自己的博客网站我想反正不会是减分项。

### 建立Hexo工程
Hexo是个nodejs框架，在安装hexo之前需要先安装npm，如果没有的话直接去npm官网安装最新版即可——[npm官网下载地址](https://www.npmjs.com/get-npm)
有了npm之后，我们在终端输入命令npm install hexo-cli -g。这是一个npm的安装命令，-g的意思是global，全局安装。成功后我们就可以用hexo-cli来创建hexo工程了。
然后在终端建立一个文件夹用来存放我们接下来的hexo工程，比如我的是~/hexo目录。在终端cd进去后，执行hexo init，初始化工程。再执行npm install来安装依赖。安装依赖可能需要点时间。
成功后我们的工程就建好了，这个时候已经有一个默认的主题landscape，一个默认的文章hello world。建立新文章有关的命令大家可以去[hexo官网](https://hexo.io/zh-cn/docs/writing)去学习，这里暂时先讲建站部署。
现在我们可以执行hexo server命令来测试一下是否能运行成功，用浏览器localhost:4000看看能不能看到我们的博客网站。
如果能看到的话说明我们的网站已经搭建好了。
<!-- more -->
### 部署到GithubPage
我们的网站虽然运行成功了，可是只能在本地访问毫无意义啊。部署到服务器的话需要先买服务器，还要备案，还要学习服务器部署网站的知识，想想就很凌乱了。
不要担心，我们可以一秒把网站部署到Github Page上，先登录[Github网站](https://github.com/)。Github是个开源代码的地方，很多公司、组织会把代码开源在这里，让大家都能看到，对个人当然也是可以在上面开源代码的。实际上Github不止可以开源代码，任何值得分享的东西都可以放在上面，比如我们的博客。没注册的先注册一下，然后跟着它的教程学一下基本的git+github流程。
注册好之后我们点击Start a project来创建一个Github Page项目，名字起为xxx.github.io，注意在创建的时候选中Initialize this repository with a README，否则等会还要上传点东西才能在放在page里。
然后进入我们创建好的项目仓库点击setting，![使用Github Page](https://upload-images.jianshu.io/upload_images/5586297-ef630def035a370c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
往下拉看到Github Pages选项，source选择master branch，然后点击save。
然后我们编辑hexo工程根目录下的_config.yml配置文件，在最下面加入如下一段：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: ssh://git@github.com/ZhangYeSong/zhangyesong.github.io
  branch: master
```
其中repo地址在我们github仓库网页上点击Clone or download可以看到如下所示：
![仓库地址](https://upload-images.jianshu.io/upload_images/5586297-1de550703397f443.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这些都搞定后，在终端hexo项目目录执行hexo deploy来试试能不能部署。
```
▶ hexo deploy
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
On branch master
nothing to commit, working tree clean
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
Everything up-to-date
Branch master set up to track remote branch master from ssh://git@github.com/ZhangYeSong/zhangyesong.github.io.
INFO  Deploy done: git
```
结束后我们在浏览器打开xxx.github.io看看有没有内容，有内容的话我们就成功啦！

### 使用阿里云域名
使用github的域名不满意咋办，没关系Github Page可以指定我们自己的域名。
这里我使用的是阿里云上买的域名，没有的话先上去买一个，然后进入域名解析界面，如下：
![阿里云域名解析](https://upload-images.jianshu.io/upload_images/5586297-0419c33ccd3da814.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里我只把blog开头的域名指向我们这个github page页面，需要www的话就把blog改成www，添加之后可能需要点时间才能生效，可以先去吃个饭。
然后我们在github上面也要改，进入setting界面，还是Github Pages选项，如下：
![指定域名](https://upload-images.jianshu.io/upload_images/5586297-08d1d193f1b57986.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在Custom domain填上我们的阿里云域名，点击save，最好把Enforce HTTPS也点上。
如果阿里云域名解析生效的话，我们就可以用我们自己的域名登录我们的个人博客网站啦！

### 把hexo项目也上传到github吧
现在还有个问题，我们想在其他电脑设备上继续写我们的博客咋办？很简单，把我们的hexo项目也上传到github就行了，在github上再建一个仓库，把我们本地的hexo项目上传上去，相当于是我们网站的源码了。
这样我们每次写一篇文章，先把hexo项目push下，再deploy一下，github上会有两笔commit，是不是很有心机～

还有个重要的问题，你会发现我们每次hexo deploy的时候，域名解析都失效了，需要重现配置解析，这是因为每次deploy后我们的CNAME文件被删除了，我们进入hexo项目根目录，进入public文件夹，新建一个CNAME文件，里面填上我们的域名，比如我的是blog.zhangyesong.com，保存。然后我们deploy的时候会加上这个CNAME域名解析，就不需要每次都要重新配置了。

### 其他
简单讲下，怎样写博客，有的小伙伴很懵逼，我的博客到底在哪里写？其实hexo只是个web工程，我们每次hexo new <博客名称>会生成一个博客名称.md文件，我们在这个markdown文件里编写博客就行了。至于你用啥编辑器去写就随意了，可以百度搜索markdown编辑器。至于markdown语法，不要害怕，很容易就学会了，而且我保证你肯定会爱上它，它是一个让人专心写内容不需要care样式的语法，学了它你就再也不想用word之类的重型文档编辑器了。
时间关系，这里先讲这么多，其实想把博客进一步弄好，还要学习很多web知识的，或者选一个别人做好的主题就行了。
最后，欢迎大家访问我的博客：blog.zhangyesong.com
