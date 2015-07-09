title: Docker 入门
date: 2015-07-09 10:52:27
tags: [Docker, Linux, Cloud, PAAS, System admin, Devlopper]
categories: Linux
---
这篇文章中，我将阐述什么是Docker，能用来做什么。并且给出我在开发和系统运维的过程中遇到的问题的解决方案。大概去年年底到今年，因为公司的项目原因，就听说了Docker这门技术，当时也去了解了一下，但是不甚理解。这几天正好项目没有太多的事情，就系统的学习了一下Docker这门技术。
<!--more-->

### 我为什么要学习Docker
在公司之前的项目中，经常遇到这样的问题：某个功能在我开发的机器上可以工作，到了其他开发人员的机器上就不能工作，或者我们已经开发好的系统部署到服务器又出现了这样那样的问题。

上述问题终归为软件系统运行的环境依赖问题，因为每个人的开发平台环境各异，服务器与本地环境也可能不相同，所以每次部署迁移的时候就漏洞百出，顾此失比。就拿部署本地开发版的系统到服务器上来说，最原始的方法就是拷贝代码到服务器，然后手动在服务器上配置，可能聪明点的Programmer可能会写自动化脚本以加快部署速度。后来又出现了各种自动化的系统管理工具，但是此类工具仍需结合你写的逻辑代码才能完成部署任务。在后来就是使用虚拟机，直接将虚拟机镜像拷贝到服务器启动即可。

但是即使最简单的虚拟机方法也是有巨大的缺点的，因为，虚拟机太消耗资源，启动几个虚拟机后，服务器就会很卡。而Docker是介于虚拟机与自动化软件配置系统之间的，没有虚拟机的巨大消耗，也不用书写部署代码，而且使用起来和GitHub很相似。

### Docker是什么
Docker使用LXC（Linux Container）技术，实现对应用环境的封装。所有开发的代码环境放置在Docker的Container中，然后其它系统就可以通过Container来部署系统。关于虚拟机和Docker的对比，请参考下图：
![vm.png](/img/vm.png)
![dockerc.png](/img/dockerc.png)
所以，简要的说，Docker是：
>An open platform for distributed applications for developers and sysadmins


### 如何使用Docker
在讲的多，不如亲自动手实践一下，这里我就不一步一步说明了，而是推荐[Get Started with Docker for Linux](http://docs.docker.com/linux/started/)，跟着教程里面的说明一步一步实践一下即可理解Docker。

#### 参考资料
[Docker到底是什么？为什么它这么火！](http://cloud.51cto.com/art/201410/453718.htm)  
[开发者可以使用Docker做什么？](http://dockone.io/article/378)  
[我们究竟要用Docker做什么](http://blog.2baxb.me/archives/1136)  
[全面了解Docker](https://github.com/DeanXu/Docker-introduce/blob/master/README.md)  
[CoreOS 实战：CoreOS 及管理工具介绍](http://www.infoq.com/cn/articles/what-is-coreos)  
[为什么COREOS和DOCKER的分手是命中注定的](http://blog.qiniu.com/archives/761)  
[How To Install and Use Docker: Getting Started](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started)  
[An Introduction to Docker by Instructor of O’Reilly’s Docker Tutorial](https://www.codementor.io/docker/tutorial/what-is-docker-tutorial-andrew-baker-oreilly)  
[Docker：利用Linux容器实现可移植的应用部署](http://www.infoq.com/cn/articles/docker-containers)  
[Docker的启示：使用者驱动创新](http://sunzhichao.baijia.baidu.com/article/36701)  
