---
title: TCP connection establish and teardown
date: 2017-02-08 13:53:21
tags:
---
This article gonna illustrate the basics of the procedure of tcp connection establish and teardown, also with some practical experiments.
<!--more-->

关于TCP/IP，读者可以仔细阅读Richard Stevens的著作《Unix Network Programming》《TCP/IP Illustrated》，还需要更详细的信息，请阅读RFC文档。

TCP, UDP层位于IP层上面，如下图为OSI和TCP/IP模型图：
![tcp/ip osi](http://d2r5da613aq50s.cloudfront.net/wp-content/uploads/296299.image0.jpg)

TCP的链接建立需要三次交换数据包，而链接断开需要四次数据包交换，所以分别叫做三次握手和四次挥手。如下图所示：
![tcp connection](https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/Tcp-handshake.svg/376px-Tcp-handshake.svg.png)
![tcp termination](https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Fin_de_conexi%C3%B3n_TCP.svg/300px-Fin_de_conexi%C3%B3n_TCP.svg.png)
