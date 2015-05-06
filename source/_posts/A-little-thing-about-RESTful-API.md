title: "A little thing about RESTful API"
date: 2015-04-24 09:14:33
tags: [Web development, RESTful API]
categories: [Web development]
---
在这篇文章中，我主要想讲一下RESTful API相关的知识。因为我感觉，以前写后台前台，就是随便写一个接口，然后两者协调，可以工作就行。后来参与的项目多了，觉得得有一套标准，这样前后台的开发人员就可以按照标准互相写作，就不会互相扯皮了。
<!--more-->

对于RESTful API的理解我也不是一下子就懂了，虽然现在看起来这概念是如此的清晰明了。

第一次知道RESTful API是在我开发一个视频点播网站系统的时候，因为当时的见识水平，所以没太在意。当时API的开发是很随便的，然后前台要使用，就直接问我，我就给他们个例子，双方不懂就问，互相调试一下，能工作就OK了。

后来到了公司实习，主要的工作也是后台开发，为了标准化点，查阅了一下RESTful API的资料，有因为是使用NodeJs做开发，所以对Json格式的数据比较了解。当时的感觉就是接受返回Json格式数据的接口就是RESTful API。其实这是不完全正确的。

后来自己开发Project的时候，又查阅了些许资料，才算对其有一个正式的了解。其实，要了解RESTful API, 你得懂一点HTTP的知识。

我们都知道HTTP协议里面定义了几个常用的方法，如GET， PUT， POST， DELETE等，本来这些方法本身就说明了所要做的操作类型。但是以前我们开发后台接口的时候是忽略了这一点的，所以开发出的接口一般是下面这种样子：
```bash
/users/create
/users/read
/users/update
/users/delete
```

但是若使用RESTful API格式开发上述接口，就变成了如下：
```bash
GET PUT POST DELETE /users
```
其中GET相当与之前的read方法，PUT相当于update方法，POST相当于create方法，DELETE相当于delete方法。

所以说，RESTful API是使用了HTTP方法并辅助一Json格式数据的接口书写方式。从使用经验上说，我觉得NodeJs对其支持最好，Python， Java， Go都对其有较好的支持。

当然了，RESTful API也不是这么一篇文章可以说完的，这里我也只是阐述我经常使用的部分而已。
