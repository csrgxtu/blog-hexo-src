---
title: WSGI 服务器性能分析【译】
date: 2020-03-08 19:36:07
tags: ['Web Server', 'Python', 'WSGI', 'Performance']
---
突然看到Omed Habib的文章：[A performance analysis of Python wsgi servers](https://www.appdynamics.com/blog/engineering/a-performance-analysis-of-python-wsgi-servers-part-2/)，觉得Bjoern确实可以用在我们项目中，用来替代现有Gunicorn。因此将原文翻译至此，希望更多可以帮助更多人。btw，Bjoern是一个用C和Libuv实现的轻量化WSGI服务器，各方面评测目前基本都处于领先的位置。
<!--more-->
在这个系列的第一篇文章中，我介绍了6款比较出名的WSGI Server。在本篇中，我会通过构建benchmark测试集来评测这6款server。  

### CGI和mod_pyton 
在WSGI标准化以前，CGI和mod_python就已经存在了。CGI是通过对每个请求创建一个新的Py进程来处理链接，因此比较低效。 mod_python相对于CGI，性能有提升，但是只能和Apache Http server 集成，况且目前处于inactive devolop的状态。

### WSGI Servers
因为时间关系，我们将重点关注下面的Wsgi server。所有的测试代码都已经发布在 [github](https://github.com/omedhabib/WSGI_Benchmarks)，且我们将会后期更新它。

* Bojern 的官方文档描述其为“非常快的Python WSGI Server” 并且它是“最快的，最小的，最轻量化的”。我们使用默认的配置创建了一个[app](https://gist.github.com/omedhabib/c3c8ff74ec3993740e80d7235251e73a) 来测试它。
* CherryPy 是一个WSGI server和app frawork的实现。
* Gunicorn 理念来自Ruby世界的Unicorn。其官方文档描述为“实现简单，轻量”。和前两者不同的是，Gunicorn是一个独立启动的server。
* Meinheld 称其为“高性能的WSGI兼容web服务器”。根据官方文档，我们创建了这个[测试程序](https://gist.github.com/omedhabib/d638e213af0f843580e5ca7724005ac6)
* mod_wsgi 和mod_python 一样来自同一个作者，且只能和apache http server搭配工作。
* uWSGI 是一个功能齐全的WSGI server。同样的根据默认配置我们建立了相应的测试程序。

### 压测
我们使用Docker来跑每个benchmark，这样可以环境隔离，每个测试启动的到时候都是干净的环境，不受当前sys的影响。

#### 服务器
* 独立的Docker 容器
* 2 CPU
* 512 RAM

#### Testing
* wrk, 一个现代化的HTTP压测工具
* 使用在100到10000之间的并发量随机测试一个服务
* wrk 限制使用2个system cpu
* 每个测试跑30秒且重复4遍

#### 性能数据
* wrk提供的请求数量、错误、延迟
* Docker stat工具提供的CPU、mem使用情况
* 去掉两边的极值然后取平均
* 请参考：[script](https://github.com/omedhabib/WSGI_Benchmarks/blob/master/benchmark.sh)

#### 结果数据
所有的评测数据都放在项目github目录下的summary csv文件。如果你感兴趣可以将其导入图形化工具来分析。

### QPS
下图展示了6款server的QPS情况，数据越大越好。
![request served](/img/wsgi_qps.png)

![request served bojern hidden](/img/wsgi_qps_bjoern_hidden.png)

* Bojern 是明显的胜出者
* CherryPy尽管用纯Python实现，但是其性能相当不错
* Meinheld也可以
* mod_wsgi 性能稳定，但是并没有那么出色
* Gunicorn 随着并发量增大qps下降
* uWSGI表现垫底
胜出者：Bojern

### 延迟
延迟就是从请求发出到响应收到这段时间，越短越好。
![wsgi_latency](/img/wsgi_latency.png)

* CherryPy 在并发量持续增长的情况下表现稳定，基本低于3毫秒
* Bjoern延迟比较大，但是在并发量低的情况下延迟较小
* Gunicorn 延迟一直稳定
* mod_wsgi 处于平均水平
* Meinheld 和Bjoern一样，当并发量增大，延迟增大
* uWSGI 最后一名
胜出者：CherryPy
Note: Bjoern is the second

### 内存使用情况
这里考察每种服务器使用内存情况，当然是越低越好。
![wsgi_mem](/img/wsgi_mem.png)

* Bjoern 内存占用最少，即使是10000的并发也只占用了9M
* Meinheld 和Bjoern基本相似
* Gunicorn 在增大并发量的情况下，内存占用稳定
* CherryPy 会随着并发增加占用变大
* mod_wsgi 表现稳定
* uWSGI 又是占用资源最多的
胜出者： Bjoern and Beinheld

### 错误
这里给出Web server丢弃，断开或者超时链接的情况。
![wsgi_error](/img/wsgi_error.png)

* CherryPy 基本上零错误
* Bjoern 有遇到error，但是其qps高很多
* mod_wsgi 大概有6%的错误率
* Gunicorn 在高并发的情况下，错误率大概9%
* uWSGI 大概34%的错误率
* Meinheld 错误率比较高
胜出者： CherryPy

### CPU使用情况
CPU使用率高不是坏事也不是好事，前提是提供服务稳定就好。但是在相同的情况下，使用率月底表现越好。

![wsgi_cpu_usage](/img/wsgi_cpu_usage.png)

* Bjoern 是单线程的，其实用率100%
* CherryPy 是多线程，但是因为GIL，其实用率基本150%
* Gunicorn 是多进程，占用率在150%以上
* Meinheld 和Gjoern差不多
* mod_wsgi 真正的多线程，所以基本跑满了CPU
* uWSGI 使用率较低
胜出者： None，因为模式不一样，无法比较

### 总结
根据压测结果，我们得出结论:Bjoern性能优异，uWSGI表现最差。
