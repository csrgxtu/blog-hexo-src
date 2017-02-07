---
title: shadowsocks tutorial
date: 2017-02-07 17:15:13
tags:
---
这篇文章阐述如何在Linux, Mac, Windos下使用Shadowsocks翻墙.

<!--more-->
#### Linux, Mac
Linux和Mac差不多，所以这里放一起。

* 首先安装shadowsocks
```bash
sudo pip install shadowsocks
```
* 创建配置文件ss.json
```bash
{
  "server": "ni.ma.com",
  "server_port": 6666,
  "local_address": "0.0.0.0",
  "local_port": 8087,
  "password": "whatthefuck",
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false
}
```

* 启动服务
```bash
sslocal -c ss.json
```
* 安装插件[switchomega](https://pan.baidu.com/s/1kUXN8Fl)到Chrome浏览器，Firefox类似. 直接拖拽插件到Chrome的插件选项页面。
![install switchomega](/img/switchomegachrome.png)

* 配置switchomega如下图，并保存
![switchomega](/img/switchomega.png)

* 打开新标签，在switchomega插件里面选择刚才保存的名称，输入www.google.com
![google](/img/google.png)

#### Windows
* 首先安装shadowsocks GUI客户端  
https://pan.baidu.com/s/1eRCZhku

* 打开客户端，填写相应的信息。同上面的Json数据差不多。

* 启动服务

* 安装插件[switchomega](https://pan.baidu.com/s/1kUXN8Fl)到Chrome浏览器，Firefox类似. 直接拖拽插件到Chrome的插件选项页面。
![install switchomega](/img/switchomegachrome.png)

* 配置switchomega如下图，并保存
![switchomega](/img/switchomega.png)

* 打开新标签，在switchomega插件里面选择刚才保存的名称，输入www.google.com
![google](/img/google.png)
