---
title: Microservice
date: 2016-10-20 09:40:42
tags:
---
看完了书籍Micoservice Architecture，结合着最近的架构重构，说一点儿遇到的问题，迷惑的地方．
<!--more-->

我们整体使用微服务的架构，对于所有的基本的数据模型，都是实现了REST格式的接口，然后使用了API Gateway，所有的这些都是集群部署，以实现高可用的要求．然后在Gateway集群上做了验证，权限控制，监控，日志等功能．这样每个微服务只关注自身的事情，而不用重复实现验证，日志等功能．整体上看起来特别理想．

REST要求对一个数据实体只要写出支持CRUD的接口即可，所以在实际中，我就提前将基本的针对数据的操作都写好，并且文档也写出来了．可在客户端实际开发的时候，对于每个页面模块的数据展示功能，本来可以直接调用已经存在的CRUD接口来满足页面展示功能的，可是他们想的是调用一次，然后把整个页面的数据返回，因为这样前端更有效率．我当然也可以实现一个接口返回这个页面所有的数据，但是这样的接口只是针对一个特定页面的，要是这个页面的设计更改了，接口也得跟着变，还有文档（写文档很慢），这是没有通用性质的．

with constant presure to add features and options and configurations and to ship code quickly, it's easy to neglect simplicity, even though in the long run simplicity is the key to good software.

只有上述句子能表达我的心声．下来去调研一下GraghQL是怎么回事，然后希望这个能达到聚合要求，最好这个聚合要求是由客户端去做，因为，天啊，谁知道他们都想要什么数据．
