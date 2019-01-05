---
title: Android Studio升级到2.3之后无限转圈的问题
date: 2017-04-09 16:15:19
categories:
 - Android
tags:
 - Android
---
最近Android Studio终于可以升级到2.3啦，可是很多人升级到2.3之后却发现AS报错或者一直在refresh。发现解决方法之后在此记录以下。


Android Studio更新提示
点击第一个按钮AS就会开始更新，如果没有推送的话，点击Help ---》check update会弹出来。

升级好之后打开项目发现AS一直卡在原地转圈，怎么办？


设置中选择gradle路径
我们打开设置界面，会告诉我们Gradle的地址错误。

我这里项目的Gradle版本由原来的2.14.1变成了3.2，选择3.2的版本点OK。


重新选择gradle版本
选择好之后AS还在转圈怎么办？强行关闭进程，重新启动。

重新打开之后，AS开始正常下载一些东西，不会无限转圈圈啦。等下载完之后就会正常编译了。

注意，在以前用旧版本gradle编译的项目都要手动去修改gradle目录，因为gradle已经进行了更新，不再是之前的目录啦。


gradle更新提示
另外，AS会提示升级gradle版本到3.3以上（那你给我更新个3.2是什么意思- -），点了的话如果升级不了可以去services.gradle.org/distributions里面下载3.3以上的版本。