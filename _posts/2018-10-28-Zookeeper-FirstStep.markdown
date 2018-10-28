---
layout:     post
title:      "先改一发zk吧"
date:       2018-10-28
author:     "yzgu"
header-img: "img/post-bg-20181002.jpg"
tags:
    - TBSchedule
---


## 关于源码的碎碎念

当幼畜询问资深码畜代码学习之道的时候，资深码畜通常会回答多看源码，至于如何多看，如何又能看得懂，那就是另外一个问题了，好心一点的前辈会说某某工程不错，可以看看，另外一部分好心的前辈往往会说，无他，但手熟尔。

幼畜往往在这时候就感到一种无以名状的绝望，大概类似于道理我都懂，你tm倒是给个用户说明啊。

很抱歉，这个确实没有，不过我倒是可以简单的比划一下，如何在不那么懂源码的基础上，稍加改造，搞出来一个自己想要的特性——放在文章里，就是给zk里添加tree命令，可以按照路径和深度打印出一个类似linux中tree命令结果的目录树形图的功能来。

第一步当然是阅读文档，并将代码运行起来
从git上下载到zookeeper的代码之后直接导入eclipse无法直接生效，需要在代码主目录下运行ant eclipse将工程转换成eclipse工程，转换完成后导入即可，整理工程，调整jdk版本等等，直到工程不再报错。就可以开始下一步了。

客户端分两种，对外开放的api客户端和命令行客户端，先从命令行客户端开始，查看zkcli.sh脚本，获得客户端启动主函数org.apache.zookeeper.ZooKeeperMain

zkcli启动不需要参数，直接运行后退出：
![img](/img/zookeeper/1.jpg)
发现代码中根据命令行类型有不同选择：
![img](/img/zookeeper/2.jpg)
![img](/img/zookeeper/3.jpg)
注意到命令行类型的区别，修改jlinemissing为true，根据标准输入循环获取命令。
![img](/img/zookeeper/4.jpg)
正常交互
![img](/img/zookeeper/5.jpg)
根据以下代码
![img](/img/zookeeper/6.jpg)
当history时，本次操作的内容被放入一个hashmap中
执行命令时：
![img](/img/zookeeper/7.jpg)
![img](/img/zookeeper/8.jpg)
Ls和get是最常用的两个命令，显然，客户端的核心对象是ZooKeeper

明天过来再改改吧