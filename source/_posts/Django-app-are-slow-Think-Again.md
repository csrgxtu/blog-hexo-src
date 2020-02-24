---
title: 'Django app are slow, Think Again'
date: 2020-02-24 11:59:38
tags: ['Python', 'Django', 'Async', 'Concurrency']
---
经常听到同事说Python服务性能不好，要去拥抱Go。但是细想，技术里面没有银弹，在某些场景下，他诚然没有编译型语言快。我们的服务都是一些I/O为主的，大部分的时间可能都是在等待网络I/O，如果换成任何一种编译型语言，难道就没有这种I/O等待了吗？再比如Instagram，Youtube，Dropbox等都在大规模使用Python，为什么人家的服务不会像蜗牛一样呢？自己的Py 服务性能差，我还能做什么呢？  
本篇来分析为什么目前Django的服务没有我们预想的好？能做些什么来改进？
<!--more-->