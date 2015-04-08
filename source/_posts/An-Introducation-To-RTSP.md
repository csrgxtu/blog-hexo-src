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

And what is **protocol** used for really? actually you can think that protocols are rules that made to make communication between endpoints easier and  much more crossplatform. just like our spoken languages we used to talk with each other.

if you roughly understand what **HTTP** is, then **RTSP** is the same thing. just like HTTP, RTSP is a text based protocol too, like HTTP methods, RTSP have its own method too, like **OPTIONS, DESCRIBE, SETUP, PLAY, PUASE, TEARDOWN** etc.

**RTSP** is a network control protocol designed for use in entertainment and communication systems to control streaming media servers. The protocol is used for establishing and controlling media sessions between end points, for example, *play* and *puase*.

### How it works
To see how it works, I will show you an example. just like HTTP has Apache and Nginx etc, rtsp has its implementation too. but here i am using a simple RTSP server implementation running on Android, namely [libstreaming](https://github.com/fyhertz/libstreaming). and client side i am using telnet.

In the following, I am gonna show you how to use telnet connect to the rtsp server and issue the methods.

First, install the libstreaming app [example1](https://github.com/fyhertz/libstreaming-examples#example-1) on your android phone. and assume your android phone address is *192.168.10.93*.

Second, start telnet and connect to android rtsp server.
```bash
$ telnet 192.168.10.93 8086
Trying 192.168.10.93...
Connected to 192.168.10.93.
Escape character is '^]'.
```

### References
[Hypertext Transfer Protocol](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)

[Real Time Messaging Protocol](http://en.wikipedia.org/wiki/Real_Time_Messaging_Protocol)

[Real Time Streaming Protocol](http://www.informit.com/articles/article.aspx?p=169578&seqNum=3)

[Wiki - Real Time Streaming Protocol](http://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol)
