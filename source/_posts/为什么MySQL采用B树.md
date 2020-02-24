---
title: 为什么MySQL采用B树
date: 2020-02-24 12:17:33
tags: ['MySQL', 'DB']
---
以前只有MySQL, 后来出了很多各种类型的数据库，比如Mongodb，Influxdb，cassandra等，不管哪种数据库，都需要使用索引算法来快速搜索数据。那么我们来看看MySQL使用索引的算法。

<!--more-->