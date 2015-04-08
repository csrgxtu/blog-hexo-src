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

Third, after connected to RTSP Server, first see what opeations it support.
```bash
OPTIONS rtsp://192.168.10.93:1234 RTSP/1.0
CSeq: 1

RTSP/1.0 200 OK
Server: MajorKernelPanic RTSP Server
Cseq: 1
Content-Length: 0
'Public: DESCRIBE,SETUP,TEARDOWN,PLAY,PAUSE'
```
You can see from the response of the RTSP Server, there are **DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE**

Fourth, let RTSP Server describe the streaming is has.
```bash
DESCRIBE rtsp://192.168.10.93:1234 RTSP/1.0
CSeq: 2

RTSP/1.0 200 OK
Server: MajorKernelPanic RTSP Server
Cseq: 2
Content-Length: 467
Content-Base: 192.168.10.93:8086/
'Content-Type: application/sdp'

v=0
o=- 0 0 IN IP4 192.168.10.93
s=Unnamed
i=N/A
c=IN IP4 192.168.10.95
t=0 0
a=recvonly
'm=audio 5004 RTP/AVP 96'
a=rtpmap:96 mpeg4-generic/8000
a=fmtp:96 streamtype=5; profile-level-id=15; mode=AAC-hbr; config=1588; SizeLength=13; IndexLength=3; IndexDeltaLength=3;
a=control:trackID=0
'm=video 5006 RTP/AVP 96'
a=rtpmap:96 H264/90000
a=fmtp:96 packetization-mode=1;profile-level-id=428014;sprop-parameter-sets=Z0KAFNoFB+Q=,aM4G4g==;
a=control:trackID=1
```
In this step, the server returned the streaming information in sdp format, as you can see audio and video information.

Fifth, now you know the streaming information, now setup the transport protocol and port information.
```bash
SETUP rtsp://192.168.10.93:1234/trackID=1 RTSP/1.0
CSeq: 3
Transport: RTP/AVP;unicast;client_port=8000-8001

RTSP/1.0 200 OK
Server: MajorKernelPanic RTSP Server
Cseq: 3
Content-Length: 0
Transport: RTP/AVP/UDP;unicast;destination=192.168.10.95;client_port=8000-8001;'server_port=39000-35968';ssrc=46a81ad7;mode=play
'Session: 1185d20035702ca'
Cache-Control: no-cache
```
In the response, the server will return the Session ID *1185d20035702ca*.

Sexth, now start to play
```bash
PLAY rtsp://192.168.10.93:1234 RTSP/1.0
CSeq: 4
Session: 1185d20035702ca

RTSP/1.0 200 OK
Server: MajorKernelPanic RTSP Server
Cseq: 4
Content-Length: 0
RTP-Info: url=rtsp://192.168.10.93:1234/trackID=1;seq=0
Session: 1185d20035702ca
```

Seventh, Puase the streaming
```bash
PAUSE rtsp://192.168.10.93:1234 RTSP/1.0
CSeq: 5
Session: 1185d20035702ca

RTSP/1.0 200 OK
Server: MajorKernelPanic RTSP Server
Cseq: 5
Content-Length: 0
```

Finally, teardown the streaming
```bash
TEARDOWN rtsp://192.18.10.93:1234 RTSP/1.0
CSeq: 6
Session: 1185d20035702ca

RTSP/1.0 200 OK
Cseq: 6
```

### RTSP Commands
**OPTIONS**  
An OPTIONS request returns the request types the server will accept.
```bash
C->S:  OPTIONS rtsp://example.com/media.mp4 RTSP/1.0
       CSeq: 1
       Require: implicit-play
       Proxy-Require: gzipped-messages

S->C:  RTSP/1.0 200 OK
       CSeq: 1
       Public: DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE
```

**DESCRIBE**  
A DESCRIBE request includes an RTSP URL (rtsp://...), and the type of reply data that can be handled. This reply includes the presentation description, typically in Session Description Protocol (SDP) format. Among other things, the presentation description lists the media streams controlled with the aggregate URL. In the typical case, there is one media stream each for audio and video.
```bash
C->S: DESCRIBE rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 2

S->C: RTSP/1.0 200 OK
      CSeq: 2
      Content-Base: rtsp://example.com/media.mp4
      Content-Type: application/sdp
      Content-Length: 460

      m=video 0 RTP/AVP 96
      a=control:streamid=0
      a=range:npt=0-7.741000
      a=length:npt=7.741000
      a=rtpmap:96 MP4V-ES/5544
      a=mimetype:string;"video/MP4V-ES"
      a=AvgBitRate:integer;304018
      a=StreamName:string;"hinted video track"
      m=audio 0 RTP/AVP 97
      a=control:streamid=1
      a=range:npt=0-7.712000
      a=length:npt=7.712000
      a=rtpmap:97 mpeg4-generic/32000/2
      a=mimetype:string;"audio/mpeg4-generic"
      a=AvgBitRate:integer;65790
      a=StreamName:string;"hinted audio track"
```

**SETUP**  
A SETUP request specifies how a single media stream must be transported. This must be done before a PLAY request is sent. The request contains the media stream URL and a transport specifier. This specifier typically includes a local port for receiving RTP data (audio or video), and another for RTCP data (meta information). The server reply usually confirms the chosen parameters, and fills in the missing parts, such as the server's chosen ports. Each media stream must be configured using SETUP before an aggregate play request may be sent.
```bash
C->S: SETUP rtsp://example.com/media.mp4/streamid=0 RTSP/1.0
      CSeq: 3
      Transport: RTP/AVP;unicast;client_port=8000-8001

S->C: RTSP/1.0 200 OK
      CSeq: 3
      Transport: RTP/AVP;unicast;client_port=8000-8001;server_port=9000-9001
      Session: 12345678
```

**PLAY**  
A PLAY request will cause one or all media streams to be played. Play requests can be stacked by sending multiple PLAY requests. The URL may be the aggregate URL (to play all media streams), or a single media stream URL (to play only that stream). A range can be specified. If no range is specified, the stream is played from the beginning and plays to the end, or, if the stream is paused, it is resumed at the point it was paused.
```bash
C->S: PLAY rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 4
      Range: npt=5-20
      Session: 12345678

S->C: RTSP/1.0 200 OK
      CSeq: 4
      Session: 12345678
      RTP-Info: url=rtsp://example.com/media.mp4/streamid=0;seq=9810092;rtptime=3450012
```

**PAUSE**  
A PAUSE request temporarily halts one or all media streams, so it can later be resumed with a PLAY request. The request contains an aggregate or media stream URL. A range parameter on a PAUSE request specifies when to pause. When the range parameter is omitted, the pause occurs immediately and indefinitely.
```bash
C->S: PAUSE rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 5
      Session: 12345678

S->C: RTSP/1.0 200 OK
      CSeq: 5
      Session: 12345678
```

**RECORD**  
This method initiates recording a range of media data according to the presentation description. The time stamp reflects start and end time(UTC). If no time range is given, use the start or end time provided in the presentation description. If the session has already started, commence recording immediately. The server decides whether to store the recorded data under the request URI or another URI. If the server does not use the request URI, the response should be 201 and contain an entity which describes the states of the request and refers to the new resource, and a Location header.
```bash
C->S: RECORD rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 6
      Session: 12345678

S->C: RTSP/1.0 200 OK
      CSeq: 6
      Session: 12345678
```

**ANNOUNCE**  
The ANNOUNCE method serves two purposes:  
When sent from client to server, ANNOUNCE posts the description of a presentation or media object identified by the request URL to a server. When sent from server to client, ANNOUNCE updates the session description in real-time. If a new media stream is added to a presentation (e.g., during a live presentation), the whole presentation description should be sent again, rather than just the additional components, so that components can be deleted.
```bash
C->S: ANNOUNCE rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 7
      Date: 23 Jan 1997 15:35:06 GMT
      Session: 12345678
      Content-Type: application/sdp
      Content-Length: 332

      v=0
      o=mhandley 2890844526 2890845468 IN IP4 126.16.64.4
      s=SDP Seminar
      i=A Seminar on the session description protocol
      u=http://www.cs.ucl.ac.uk/staff/M.Handley/sdp.03.ps
      e=mjh@isi.edu (Mark Handley)
      c=IN IP4 224.2.17.12/127
      t=2873397496 2873404696
      a=recvonly
      m=audio 3456 RTP/AVP 0
      m=video 2232 RTP/AVP 31

S->C: RTSP/1.0 200 OK
      CSeq: 7
```

**TEARDOWN**  
A TEARDOWN request is used to terminate the session. It stops all media streams and frees all session related data on the server.
```bash
C->S: TEARDOWN rtsp://example.com/media.mp4 RTSP/1.0
      CSeq: 8
      Session: 12345678

S->C: RTSP/1.0 200 OK
      CSeq: 8
```

### References
[Hypertext Transfer Protocol](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)

[Real Time Messaging Protocol](http://en.wikipedia.org/wiki/Real_Time_Messaging_Protocol)

[Real Time Streaming Protocol](http://www.informit.com/articles/article.aspx?p=169578&seqNum=3)

[Wiki - Real Time Streaming Protocol](http://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol)
