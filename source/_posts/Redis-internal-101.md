---
title: Redis internal 101 --[Draft]
date: 2020-02-24 12:18:00
tags: ['Cache', 'Redis']
---
项目中经常使用Redis作为cache方案，但是你知道为什么Redis作为一种缓存速度如此快？为什么ttl能expire掉你的key？本文来分析这些东西。

<!--more-->