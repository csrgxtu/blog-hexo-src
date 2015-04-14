title: "CORS 跨域资源共享"
date: 2015-04-14 12:45:15
tags: [http, cors, XMLHttpRequest, JavaScript]
categories: [JavaScript, HTTP]
---
本文将对**CORS** -- Cross Origin Resource Sharing即跨域资源共享做简单的阐述。你将了解到CORS的基本情况以及与其相关的一些知识。

作为开发人员，一般情况下不会接触到CORS，因为大多数情况下，你只要后台写好API，前台通过AJAX异步访问即可。可是有的时候，前台页面要展示的数据并不是来自当前域后台，而是由其它域的后台提供。这个时候要想获取其它域的后台数据，就必须使用**CORS**技术了。
<!--more-->
