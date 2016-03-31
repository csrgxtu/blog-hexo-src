---
title: Git Github 简单使用
date: 2016-03-31 08:48:16
tags: ['git', 'github']
---
这里简单阐述如何使用Git和Github作为你的开发工具链。Git是Linus Tovards不满私有代码库的限制而开发出来的一套分布式的源代码管理工具，作为个人用户来说主要是可以记录每次代码都修改了什么，然后可以回退等操作。而Github是免费的托管代码的平台，可以使用Git向Github提交代码。若不使用Github，也可以使用开源的Gitlab。
<!--more-->

### 如何使用Git和Github
这里通过穿件一个repository并且从本地上传代码为例子阐述。步骤如下：

#### 在本机安装Git工具

若是在Linux下，可以直接使用apt包管理器安装，如下：
```bash
sudo apt-get install git
```
若是在Windows下，可以下载相应的安装包，点击安装即可。[下载链接](https://git-scm.com/downloads)

安装完成后，在命令行输入git可以看到如下输出：
```bash
$ git --help
usage: git [--version] [--help] [-C <path>] [-c name=value]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p|--paginate|--no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]

The most commonly used git commands are:
   add        Add file contents to the index
   bisect     Find by binary search the change that introduced a bug
```
出现上述输出，说明安装成功。

#### 在Github上注册账号
在浏览器上打开[Github](https://github.com/),然后注册一个账号。

#### 在Github上登录然后创建repo
使用注册的账号登陆[Github](https://github.com/),首次登录，你的账号是没有repo的，所以需要按照提示创建一个新的repo。如下图：
![create repo](/img/createrepo.png)
![create detail](/img/createdetail.png)
![repo](/img/repo.png)

#### 使用Git将repo复制到本地
拷贝https的git地址，然后使用git工具克隆到本地：
```bash
git clone https://github.com/csrgxtu/gitgithub.git
```

#### 在本地修改代码，然后上传
进入克隆下来的文件夹，创建一个文件，然后提交上传。
```bash
cd gitgithub
touch demo.txt
git status # 查看更改
git add demo.txt # 将demo.txt添加到git跟踪里面
git commit -m "demo.txt" # 将demo.txt提交到本地代码库
git push # 将本地代码库sync到github
```

#### 在Github上查看你的本地修改
登陆你的github的repo的主页，查看修改后的文件。发现demo.txt已经同步到github。
![demo.txt.png](/img/demo.txt.png)

### 总结
Git的使用还有好多细节，比如创建分支用于多人协作，如何解决冲突，如何给代码打标签，如何release等等，善用Google。
