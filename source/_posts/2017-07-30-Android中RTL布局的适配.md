---
title: Android中RTL布局的适配
date: 2017-07-30 16:13:07
categories:
 - Android
tags:
 - Android
---
> 本文为本人原创，转载请注明作者和出处。

因为业务原因，产品需要适配阿拉伯语语、波斯语、乌尔都语。而这三种语言都是从右到左排布的。为了照顾这部分客户的需求需要对整体布局也进行相应的适配。不得不说谷歌真的是很博爱，Android自身留了大量的API以供适配RTL（从右到左）布局。可能有这种需求的人比较少，文章也比较少，特此记录以帮助其他需要的人。


* **布局属性**
Android在4.2的时候为了支持RTL布局引入了marginStart、marginEnd等属性来替代marginLeft和marginRight等属性。如果你的APP需要适配4.2以下的版本需要同时设置这两种属性。使用XXXStart、XXXEnd属性来代替XXXLeft、XXXRight属性，无论是正常的布局还是RTL布局都可以正常显示，这样就不用专门写一个layout来适配RTL布局了。

* **\u200f转义符**
有的时候英语和阿拉伯语、波斯语混用会出现这种情况：英语从左往右排布，而阿拉伯语从右往左排布。这种情况对于阿拉伯语都用户肯定是混乱的。在字符串首尾加上\u200f转义符可以强行让字符串从右往左显示。

* **drawable-ldrtl**
图片也需要适配RTL布局怎么办？和适配其他分辨率之类的一样，将RTL布局需要显示的图片放在drawable-ldrtl文件夹下就可以让图片也适配RTL布局了。

* **autoMirrored属性**
在drawable的xml文件中有autoMirrored这一属性，将其设置为true，就可以让drawable在RTL布局下进行反转。相比于上面的方法这种方法更节省APK空间一点。

* **textDirection属性**
设置textDirection="locale"，可以让文字自动适配是不是RTL布局。也可以设置为rtl或ltr让文字永远从右到左或从左到右显示。

* **根据语言进行适配**
还有其他情况的话可以获取当前系统的语言来进行适配。代码如下：
```
String language = Locale.getDefault().getLanguage();
if (language.equals("ar") || language.equals("fa") || language.equals("ur")) {
    //RTL布局下的适配修改
}
```
