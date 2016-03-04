title: "Learning 30 Technologies in 30 Days: A Developer Challenge - bower "
date: 2016-03-04 17:35:35
tags:
---
### 什么是Bower？
Bower是一个客户端技术的软件包管理器，它可用于搜索、安装和卸载如JavaScript、HTML、CSS之类的网络资源。其他一些建立在Bower基础之上的开发工具，如YeoMan和Grunt，这个会在以后的文章中介绍。
<!--more-->

### 为什么我会在意Bower？
* 节省时间。为什么要学习Bower的第一个原因，就是它会为你节省寻找客户端的依赖关系的时间。每次我需要安装jQuery的时候，我都需要去jQuery网站下载包或使用CDN版本。但是有了Bower，你只需要输入一个命令，jquery就会安装在本地计算机上，你不需要去记版本号之类的东西，你也可以通过Bower的info命令去查看任意库的信息。

* 脱机工作。Bower会在用户主目录下创建一个.bower的文件夹，这个文件夹会下载所有的资源、并安装一个软件包使它们可以离线使用。如果你熟悉Java，Bower即是一个类似于现在流行的Maven构建系统的.m2仓库。每次你下载任何资源库都将被安装在两个文件夹中 —— 一个在的应用程序文件夹，另一个在用户主目录下的.bower文件夹。因此，下一次你需要这个仓库时，就会用那个用户主目录下.bower中的版本。

* 可以很容易地展现客户端的依赖关系。你可以创建一个名为bower.json的文件，在这个文件里你可以指定所有客户端的依赖关系，任何时候你需要弄清楚你正在使用哪些库，你可以参考这个文件。

* 让升级变得简单。假设某个库的新版本发布了一个重要的安全修补程序，为了安装新版本，你只需要运行一个命令，bower会自动更新所有有关新版本的依赖关系。

### 前提准备
为了安装bower，你首先需要安装如下文件：
* Node：下载最新版本的node.js
* NPM：NPM是node程序包管理器。它是捆绑在nodejs的安装程序上的，所以一旦你已经安装了node，NPM也就安装好了。
* Git：你需要从git仓库获取一些代码包。

### 安装Bower
一旦你已经安装了上面所说的所有必要文件，键入以下命令安装Bower：
```bash
npm install -g bower
```
这行命令是Bower的全局安装，-g 操作表示全局。

### 开始使用Bower
安装完bower之后就可以使用所有的bower命令了。可以键入help 命令来查看bower可以完成那些操作，如下：
```bash
$ bower help
Usage:

    bower <command> [<args>] [<options>]

Commands:

    cache                   Manage bower cache
    help                    Display help information about Bower
    home                    Opens a package homepage into your favorite browser
    info                    Info of a particular package
    init                    Interactively create a bower.json file
    install                 Install a package locally
    link                    Symlink a package folder
    list                    List local packages
    lookup                  Look up a package URL by name
    prune                   Removes local extraneous packages
    register                Register a package
    search                  Search for a package by name
    update                  Update a local package
    uninstall               Remove a local package

Options:

    -f, --force             Makes various commands more forceful
    -j, --json              Output consumable JSON
    -l, --log-level         What level of logs to report
    -o, --offline           Do not hit the network
    -q, --quiet             Only output important information
    -s, --silent            Do not output anything, besides errors
    -V, --verbose           Makes output more verbose
    --allow-root            Allows running commands as root
```

### 包的安装
Bower是一个软件包管理器，所以你可以在应用程序中用它来安装新的软件包。举例来看一下来如何使用Bower安装JQuery，在你想要安装该包的地方创建一个新的文件夹，键入如下命令：
```bash
bower install jquery
```

上述命令完成以后，你会在你刚才创建的目录下看到一个bower_components的文件夹，其中目录如下：
```bash
$ tree bower_components/
bower_components/
└── jquery
    ├── README.md
    ├── bower.json
    ├── component.json
    ├── composer.json
    ├── jquery-migrate.js
    ├── jquery-migrate.min.js
    ├── jquery.js
    ├── jquery.min.js
    ├── jquery.min.map
    └── package.json

1 directory, 10 files
```

### 包的使用
现在就可以在应用程序中使用jQuery包了，在jQuery里创建一个简单的html5文件：
```html
<!doctype html>
<html>
<head>
    <title>Learning Bower</title>
</head>
<body>

<button>Animate Me!!</button>
<div style="background:red;height:100px;width:100px;position:absolute;">
</div>

<script type="text/javascript" src="bower_components/jquery/jquery.min.js"></script>
<script type="text/javascript">

    $(document).ready(function(){
        $("button").click(function(){
            $("div").animate({left:'250px'});
        });
    });
</script>
</body>
</html>

```
正如你所看到的，你刚刚引用jquery.min.js文件，现阶段完成。

### 所有包的列表
如果你想找出所有安装在应用程序中的包，可以使用list命令：
```bash
$ bower list
bower check-new     Checking for new versions of the project dependencies..
blog /Users/shekhargulati/day1/blog
└── jquery#2.0.3 extraneous
```

### 包的搜索
假如你想在你的应用程序中使用twitter的bootstrap框架，但你不确定包的名字，这时你可以使用search 命令：
```bash
$ bower search bootstrap
Search results:

    bootstrap git://github.com/twbs/bootstrap.git
    angular-bootstrap git://github.com/angular-ui/bootstrap-bower.git
    sass-bootstrap git://github.com/jlong/sass-twitter-bootstrap.git
```

### 包的信息
如果你想看到关于特定的包的信息，可以使用info 命令来查看该包的所有信息：
```bash
$ bower info bootstrap
bower bootstrap#*           not-cached git://github.com/twbs/bootstrap.git#*
bower bootstrap#*              resolve git://github.com/twbs/bootstrap.git#*
bower bootstrap#*             download https://github.com/twbs/bootstrap/archive/v3.0.0.tar.gz
bower bootstrap#*              extract archive.tar.gz
bower bootstrap#*             resolved git://github.com/twbs/bootstrap.git#3.0.0

{
  name: 'bootstrap',
  version: '3.0.0',
  main: [
    './dist/js/bootstrap.js',
    './dist/css/bootstrap.css'
  ],
  ignore: [
    '**/.*'
  ],
  dependencies: {
    jquery: '>= 1.9.0'
  },
  homepage: 'https://github.com/twbs/bootstrap'
}

Available versions:
  - 3.0.0
  - 3.0.0-rc1
  - 3.0.0-rc.2
  - 2.3.2
 .....
```
如果你想得到单个包的信息，也可以使用info 命令：
```bash
$ bower info bootstrap#3.0.0
bower bootstrap#3.0.0           cached git://github.com/twbs/bootstrap.git#3.0.0
bower bootstrap#3.0.0         validate 3.0.0 against git://github.com/twbs/bootstrap.git#3.0.0

{
  name: 'bootstrap',
  version: '3.0.0',
  main: [
    './dist/js/bootstrap.js',
    './dist/css/bootstrap.css'
  ],
  ignore: [
    '**/.*'
  ],
  dependencies: {
    jquery: '>= 1.9.0'
  },
  homepage: 'https://github.com/twbs/bootstrap'
}
```
### 包的卸载
卸载包可以使用uninstall 命令：
```bash
$ bower uninstall jquery
```

### bower.json文件的使用
bower.json文件的使用可以让包的安装更容易，你可以在应用程序的根目录下创建一个名为“bower.json”的文件，并定义它的依赖关系。使用bower init 命令来创建bower.json文件：
```bash
$ bower init
[?] name: blog
[?] version: 0.0.1
[?] description:
[?] main file:
[?] keywords:
[?] authors: Shekhar Gulati <shekhargulati84@gmail.com>
[?] license: MIT
[?] homepage:
[?] set currently installed components as dependencies? Yes
[?] add commonly ignored files to ignore list? Yes
[?] would you like to mark this package as private which prevents it from being accidentally published to the registry? No

{
  name: 'blog',
  version: '0.0.1',
  authors: [
    'Shekhar Gulati <shekhargulati84@gmail.com>'
  ],
  license: 'MIT',
  ignore: [
    '**/.*',
    'node_modules',
    'bower_components',
    'test',
    'tests'
  ],
  dependencies: {
    jquery: '~2.0.3'
  }
}

[?] Looks good? Yes
```
可以查看该文件：
```json
{
  "name": "blog",
  "version": "0.0.1",
  "authors": [
    "Shekhar Gulati <shekhargulati84@gmail.com>"
  ],
  "license": "MIT",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "jquery": "~2.0.3"
  }
}
```
注意看，它已经加入了jQuery依赖关系。  
现在假设也想用twitter bootstrap，我们可以用下面的命令安装twitter bootstrap并更新bower.json文件：
```bash
$ bower install bootstrap --save
```
它会自动安装最新版本的bootstrap并更新bower.json文件：
```json
{
  "name": "blog",
  "version": "0.0.1",
  "authors": [
    "Shekhar Gulati <shekhargulati84@gmail.com>"
  ],
  "license": "MIT",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "jquery": "~2.0.3",
    "bootstrap": "~3.0.0"
  }
}
```
这就是今天的学习，希望能让你对bower有个足够的了解，最好可以自己尝试一下。

转载自[SegmentFault](https://segmentfault.com/a/1190000000349555)
英文原文[bower](https://blog.openshift.com/learning-30-technologies-in-30-days-a-developer-challenge/)
