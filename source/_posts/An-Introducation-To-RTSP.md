title: "An Introducation To RTSP"
date: 2015-04-08 13:34:18
tags: [RTSP, Video Streaming]
categories: Linux
---
In this post, I am gonna to introduce you a streaming protocol **RTSP**(Real Time Streaming Protocol). cause to get myself understand what it is takes me a lot energy, so here you will see a simpler but not too simpler introduction.

Through the introduction, you will see what it is used for and do some experiments with it.
<!--more-->

### What is RTSP
Before talking about RTSP, I want to talk about **HTTP** -- Hyper Text Transfer Protocol. cause most of us are much familiar with HTTP rather than RTSP. every time we enter a word in Google Search, the browser will start an HTTP session with web server through HTTP, even through you don't know the undergoing actions.

[HTTP](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is a text based protocol that determines how the browser and web server communicate with each other. if you know something about computer networks, you will know that there are a lot protocols there, like HTTPS, FTP, RTMP, SSH, TCP/UDP etc...

You may wonder *text based*, yeah, text based means the protocol is opeate through regular text, just like Bash, you type a text string command, and system will execute that string command. here is a snippet of text based example:
```bash
HTTP: GET /index.php HTTP/1.0
BASH: $who
```

The main thing you will remember about HTTP is its methods, like **GET, PUT, POST, DELETE, HEAD, OPTIONS** etc, and recent years, their is a jargon called **RESTful API**, it is based on HTTP and these methods.


### How it works

### References
[Hypertext Transfer Protocol](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)

[Real Time Messaging Protocol](http://en.wikipedia.org/wiki/Real_Time_Messaging_Protocol)

[Real Time Streaming Protocol](http://www.informit.com/articles/article.aspx?p=169578&seqNum=3)

[Wiki - Real Time Streaming Protocol](http://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol)
