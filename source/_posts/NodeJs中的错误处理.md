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
因为NodeJs采用单线程，异步IO的处理方式，服务器的主体程序是在一个JavaScript主线程中通过异步的方式执行的，所以一旦编写的程序哪一个部分出错，则可能导致服务器崩溃或者后续的操作不可预测，最后导致整个服务器崩溃掉。而多线程模式的服务器一般不存在这个问题，因为顶多是某一个线程崩溃掉，整个服务器还是在线的。所以使用NodeJs构建Web服务器对Programmer要求更高。

### 错误的类别
一般来说，程序中会出现的错误分为两种，一种是Operational错误，一种是Programmer错误。
Operational错误指即使是程序书写正确，也会在运行的时候遇到的问题，如：
- failed to connect to server
- failed to resolve hostname
- invalid user input
- request timeout
- server returned a 500 response
- socket hang-up
- system is out of memory
- ...

以上这些错误只是简单的举个例子，在实际的问题中，我们可能遇到各种各样的错误，而这些错误一般不是因为我们程序书写错误，而是由于我们所调用的类库出错而抛出的。

Programmer错误指程序书写有误，往往我们可以通过修改程序修正这种错误。如：
- tried to read property of "undefined"
- called an asynchronous function without a callback
- passed a "string" where an object was expected
- passed an object where an IP address string was expected
- ...

### 一般程序处理错误的方式
以上的错误分类在各种编程语言中都会遇到，但是各种编程语言处理错误的方式却不相同。如C语言通过**函数返回值**的形式来处理错误，这是最经典朴素的方式。C++，Java，Python当然也支持最函数返回值的形式，但是侧重于使用**throw/try/catch**模式处理错误。例如在Java中你会经常需要处理IOExeption，不处理，那么你的程序有可能崩掉。而到了异步编程的世界，一般来说前面两种处理方式不起作用，例如在异步函数中你使用throw抛出错误，而在异步函数的caller中就捕捉不到抛出的异常，因为caller在准备接收异常的时候，可能throw所在的函数还没有运行，或者throw的时候，caller早已经过去了。所以说一般在异步编程中，不使用throw/try/catch。当然，为了处理异步编程中的错误，引入了**回调函数**和**eventEmitter**模式。若不了解什么是回调函数和eventEmitter，请参考[What are callbacks](https://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-callbacks)和[What are Event Emitters](https://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-event-emitters)。

### 处理Operational Error
对于一个具体的Operational Error，一般我们可以如下处理：
- **Deal with the failure directly.** Sometimes, it's clear what you have to do to handle an error. If you get an <font color="red">ENOENT</font> error trying to open a log file, maybe this is the first time the program has run on this system and you just need to create the log file first. A more interesting case might be where you're maintaining a persistent connection to a server (e.g., a database), and you get a "socket hang-up" error. This usually means either the remote side or the network flaked out, and it's frequently transient, so you'd usually deal with this by reconnecting. (This isn't the same as retrying, below, since there's not necessarily an operation going on when you get this error.)
- **Propagate the failure to your client.** If you don't know how to deal with the error, the simplest thing to do is to abort whatever operation you're trying to do, clean up whatever you've started, and deliver an error back to your client. (How to deliver that error is another question, and it's discussed below.) This is appropriate when you expect that whatever caused the error is not going to change soon. For example, if the user gave you invalid JSON, it's not going to help to try parsing it again.
- **Retry the operation. ** For errors from the network and remote services (e.g., a web service), it's sometimes useful to retry an operation that returns an error. For example, if a remote service gives a 503 (Service Unavailable error), you may want to retry in a few seconds.** If you're going to retry, you should clearly document that you may retry multiple times, how many times you'll try before failing, and how long you'll wait between retries. Also, don't assume that you should always retry an operation.** If you're several layers deep in the stack (e.g., you're being called by a client, which was called by another client, which is being driven by a human), it's usually better to fail fast and let the end client deal with retries. If every layer of the stack thinks it needs to retry on errors, the user can end up waiting much longer than they should because because each layer didn't realize that the underlying layer was also retrying.
- **Blow up.** For errors that truly can't happen, or would effectively represent programmer errors if they ever did (e.g., failed to connect to a localhost socket that's supposed to be listening in the same program), it's fine to log an error message and crash. Other errors like running out of memory effectively can't be handled in a dynamic language like JavaScript anyway, so it may be totally reasonable to crash. (That said, you can get <font color="red">ENOMEM</font> from discrete operations like <font color="red">child_process.exec</font>, and those you can reasonably handle, and you should consider doing so.) You can also blow up if there's nothing you can reasonably do about something and an administrator needs to fix things. For example, if you run out of file descriptors or don't have permission to access your configuration file, there's nothing you can do about this, and a user will have to log in and fix things anyway.
- **Log the error — and do nothing else.** Sometimes, there's nothing you can do about something, there's nothing to retry or abort, and there's also no reason to crash the program. An example might be if you're keeping track of a group of remote services using DNS and one of those services falls out of DNS. There's nothing you can do about it except log a message and proceed with the remaining services. But you should at least log something in this case. (There are exceptions to every rule. If this is something that may happen thousands of times per second, and there's nothing you can do about it, it's probably not worth logging it every time it happens. But do log it periodically.)

### 处理Programmer Error
对于Programmer Error，最好的方式就是让程序停止掉，然后修改错误。因为若继续让程序运行，那么后面就会遇到无法判断的错误。

### 总结
在实际的编码过程中，遇到的情况可能比较复杂，对于一个错误，无法判断类别。但是无论如何，你是要让程序更健壮，可控。针对程序中的错误类型，合理处理即可。
### 参考资料
[Error Handling in Node.js](https://www.joyent.com/developers/node/design/errors)

[深入浅出NodeJs](http://book.douban.com/subject/25768396/) - 第三章 异步I/O

[What-Is-Try-Catch](https://docs.nodejitsu.com/articles/errors/what-is-try-catch)

[Node.js Error Handling Patterns](http://www.nodewiz.biz/nodejs-error-handling-pattern/)

[What are callbacks](https://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-callbacks)

[Mixu's Node book](http://book.mixu.net/node/)

[What are Event Emitters](https://docs.nodejitsu.com/articles/getting-started/control-flow/what-are-event-emitters)

[Using Node's Event Module](http://code.tutsplus.com/tutorials/using-nodes-event-module--net-35941)
