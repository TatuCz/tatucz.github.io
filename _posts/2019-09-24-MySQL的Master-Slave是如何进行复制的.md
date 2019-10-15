---
layout: article
title: MySQL的Master-Slave是如何进行复制的？
mathjax: true
aside:
  toc: true
sidebar:
  nav: my-navi
---


复制方式可以是Master推送给Slave进行同步，也可以Slave定时轮询Master是否有可更新数据，有则拉取。

那么，Master和Slave的复制是"Master主动推"，还是"Slave拉"呢？

答案是Master推。

当slave启动后执行了start slave，就开始了同步，由master的dump线程来主动推，具体流程见图。

![mysql_replication](https://raw.githubusercontent.com/TatuCz/tatucz.github.io/master/screenshots/MySQL_Replication.jpg)
