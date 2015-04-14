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

### 参考文献
[Cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)

[HTTP access control (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)

[High Performance Browser Networking](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
