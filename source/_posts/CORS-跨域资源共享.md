title: "CORS 跨域资源共享"
date: 2015-04-14 12:45:15
tags: [http, cors, XMLHttpRequest, JavaScript]
categories: [JavaScript, HTTP]
---
本文将对**CORS** -- Cross Origin Resource Sharing即跨域资源共享做简单的阐述。你将了解到CORS的基本情况以及与其相关的一些知识。

作为开发人员，一般情况下不会接触到CORS，因为大多数情况下，你只要后台写好API，前台通过AJAX异步访问即可。可是有的时候，前台页面要展示的数据并不是来自当前域后台，而是由其它域的后台提供。这个时候要想获取其它域的后台数据，就必须使用**CORS**技术了。
<!--more-->

### What is CORS
**CORS**只是一种标准，一种规定了客户端，服务器如何通信的机制，不同的服务器实现不同，不同的客户端实现亦不相同。

要讲**CORS**，则必须理解XMLHttpRequest， JavaScript， DHTML等技术。

JavaScript开始叫做LiveScript，因为当时是Netscape公司和Sun公司合作开发，目标是开发一种可以在浏览器中执行的语言，高级程序员 Brendan Eich只有10天左右的时间，匆匆之下完成了编译器，语言规范的设计实现，但是后来他自己也承认了JavaScript是有很多地方不完美。又因为当时Java语言正风起云涌，所以为了避免误解重名，他将其命名为LiveScript，且LiveScript首先在火狐类浏览器中运行。至于后来为什么更名为JavaScript，这我也不知道了。

DHTML则是一种动态网页技术，其中的D代表Dynamic的意思。有了DHTML，我们就可以动态的修改HTML中的内容。这也就是我们经常能使用JavaScript动态修改网页内容的根本原因了。

XMLHttpRequest， 其实是一种在浏览器端实现的可以做Http请求的技术。具体由浏览器实现，暴露给我们的则是简单易用的API。令我惊讶的是，XMLHttpRequest最开始是在微软的Internet Explore5项目中实现的。其实，如果你使用过XMLHttpRequest，你会疑问这个和XML有什么关系？uh，应该是没有直接的关系，只是由于Alex Hopmann开发出这个技术的时候，将其放入到了Internet Explore5项目中的MSXML库中才命名为如此的缘故。**可见微软是个多么随便的公司，正好也验证了其一团糟的软件产品。**

好了，上面说了这么多，还是没有到**CORS**的故事。因为有了XMLHttpRequest的技术，所以我们可以在网页中使用JavaScript访问其它网络资源。但是在客户端与服务器交互的过程中，会产生cookies等涉及安全的东西，若不加限制，XMLHttpRequest则会引起安全漏洞。所以默认的XMLHttpRequest是使用**同源**机制的，只有在同一个域名中，XMLHttpRequest才能访问成功。若example.com中的XMLHttpRequest要访问foo.com中的资源，则会失败。

但是有时候，我们确实需要在一个域名中通过XMLHttpRequest访问另一个域名的资源。所以就有了**CORS**技术。

### 参考文献
[Cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)

[HTTP access control (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)

[High Performance Browser Networking](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)

[A Short History of JavaScript](https://www.w3.org/community/webed/wiki/A_Short_History_of_JavaScript)
