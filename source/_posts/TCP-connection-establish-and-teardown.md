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
![tcp handshake](/img/tcphandshake.png)

下面两段代码表示client连接到服务端，向服务端发送一个文本消息，服务端回复一个文本消息，服务端断开，客户端断开。
~~~~c
// client.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

void error(const char *msg)
{
    perror(msg);
    exit(0);
}

int main(int argc, char *argv[])
{
    int sockfd, portno, n;
    struct sockaddr_in serv_addr;
    struct hostent *server;

    char buffer[256];
    if (argc < 3) {
       fprintf(stderr,"usage %s hostname port\n", argv[0]);
       exit(0);
    }
    portno = atoi(argv[2]);
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
        error("ERROR opening socket");
    server = gethostbyname(argv[1]);
    if (server == NULL) {
        fprintf(stderr,"ERROR, no such host\n");
        exit(0);
    }
    bzero((char *) &serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    bcopy((char *)server->h_addr,
         (char *)&serv_addr.sin_addr.s_addr,
         server->h_length);
    serv_addr.sin_port = htons(portno);
    if (connect(sockfd,(struct sockaddr *) &serv_addr,sizeof(serv_addr)) < 0)
        error("ERROR connecting");
    printf("Please enter the message: ");
    bzero(buffer,256);
    fgets(buffer,255,stdin);
    n = write(sockfd,buffer,strlen(buffer));
    if (n < 0)
         error("ERROR writing to socket");
    bzero(buffer,256);
    n = read(sockfd,buffer,255);
    if (n < 0)
         error("ERROR reading from socket");
    printf("%s\n",buffer);
    close(sockfd);
    return 0;
}

~~~~

~~~~c
//server.c
/* A simple server in the internet domain using TCP
   The port number is passed as an argument */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

void error(const char *msg)
{
    perror(msg);
    exit(1);
}

int main(int argc, char *argv[])
{
     int sockfd, newsockfd, portno;
     socklen_t clilen;
     char buffer[256];
     struct sockaddr_in serv_addr, cli_addr;
     int n;
     if (argc < 2) {
         fprintf(stderr,"ERROR, no port provided\n");
         exit(1);
     }
     sockfd = socket(AF_INET, SOCK_STREAM, 0);
     if (sockfd < 0)
        error("ERROR opening socket");
     bzero((char *) &serv_addr, sizeof(serv_addr));
     portno = atoi(argv[1]);
     serv_addr.sin_family = AF_INET;
     serv_addr.sin_addr.s_addr = INADDR_ANY;
     serv_addr.sin_port = htons(portno);
     if (bind(sockfd, (struct sockaddr *) &serv_addr,
              sizeof(serv_addr)) < 0)
              error("ERROR on binding");
     listen(sockfd,5);
     clilen = sizeof(cli_addr);
     newsockfd = accept(sockfd,
                 (struct sockaddr *) &cli_addr,
                 &clilen);
     if (newsockfd < 0)
          error("ERROR on accept");
     bzero(buffer,256);
     n = read(newsockfd,buffer,255);
     if (n < 0) error("ERROR reading from socket");
     printf("Here is the message: %s\n",buffer);
     n = write(newsockfd,"I got your message",18);
     if (n < 0) error("ERROR writing to socket");
     close(newsockfd);
     close(sockfd);
     return 0;
}

~~~~

下图是在Ubuntu 14.04 Linux上通过client, server通信抓包的分析截图：
![connectionAndtermination.png](/img/connectionAndtermnination.png)
** 链接建立 **  
从上图可以看出，编号为276的数据包是client和server建立链接的第一SYN包，277为server对client SYN的确认以及自己的SYN包，278为client对server的SYN确认，至此，TCP链接正式建立，可以开始发送数据了。

** 发送数据 **  
编号296为client向server主动发送的一个文本消息，297为server对消息的确认。298为server想client发送的一个文本消息。

** 断开链接 **  
编号299的数据包为server发送完文本消息后主动断开链接，通过设置TCP header的FIN标志位。300为client接收到server发送过来的FIN包对其确认。client端通过调用close方法发送FIN包（编号301）给服务端，然后服务端对其确认(编号302)。
