title: "NodeJs中的错误处理"
date: 2015-03-25 13:21:38
tags: [NodeJs, Error Handling]
categories: [NodeJs, Programming]
---
这篇文章中我将要阐述在NodeJs中如何处理错误(异常)。因为在之前的项目中，整个后台和前端都是用NodeJs和JavaScript实现，在开发的过程中，经常遇到服务器崩溃，在参考了几篇文章和项目组讨论的经验后，我决定简单的总结下如何在NodeJs中处理错误，希望其它的开发者能从中吸取经验，编写出更健壮的应用系统。

这篇文章所述的错误处理不只是NodeJs的开发者可以阅读，其它语言平台的也可以阅读，因为在思考错误处理这个问题的过程中，我也是参考了其它语言平台的错误处理办法，况且，我们作为Programmer，知道的也不只是一门语言，对比着学习能更好的掌握知识。
<!--more-->

### Web服务器演变
我们知道，Web服务器最开始是以单进程模式处理客户端请求，但是若一个客户端请求会消耗大量的资源或者需要大量的服务器处理时间，则后续的请求就会因得不到满足而被丢弃掉。为了处理多用户请求的问题，Web服务器引进了多进程，多线程模式，为每个到来的客户端请求建立一个进程（线程）。例如Apache，Tomcat服务器就是多线程模式的。通常，多进程（线程）模式中，进程（线程）之间是独立的，不会共享变量之类的。例如在LAMP的配置中，每次客户端的请求到达index.php，服务端都会创建一个线程，载入index.php，在独立的PHP环境中解释并运行index.php文件。一个线程的执行不会依赖与其它线程的执行。

后来服务器端又采用了单线程异步IO的处理方式，如Nginx和NodeJs。你可能会疑问，我们的计算机一般都是多核多CPU的，为什么不是多线程异步IO的处理方式呢？因为多进程（线程）编程是痛苦的，如果你做过相关的多进程（线程）并发编程，你可能知道在编写程序的过程中维护共享变量，或者进程间通信是多么痛苦的一件事情，这样的程序极容易出错，而且很难维护。所以NodeJs的作者采用单线程，异步IO的方式来实现高并发服务器。

### NodeJs服务器的问题
因为NodeJs采用单线程，异步IO的处理方式，服务其的主体程序是在一个JavaScript主线程中通过异步的方式执行的，所以一旦编写的程序哪一个部分出错，则可能导致服务器崩溃或者后续的操作不可预测，最后导致整个服务器崩溃掉。而多线程模式的服务器一般不存在这个问题，因为顶多是某一个线程崩溃掉，整个服务器还是在线的。所以使用NodeJs构建Web服务器对Programmer要求更高。

### 参考资料
[Error Handling in Node.js](https://www.joyent.com/developers/node/design/errors)

[深入浅出NodeJs](http://book.douban.com/subject/25768396/) - 第三章 异步I/O

[What-Is-Try-Catch](https://docs.nodejitsu.com/articles/errors/what-is-try-catch)

[Node.js Error Handling Patterns](http://www.nodewiz.biz/nodejs-error-handling-pattern/)

[What are callbacks](https://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-callbacks)

[Mixu's Node book](http://book.mixu.net/node/)

[What are Event Emitters](https://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-event-emitters)

[Using Node's Event Module](http://code.tutsplus.com/tutorials/using-nodes-event-module--net-35941)
