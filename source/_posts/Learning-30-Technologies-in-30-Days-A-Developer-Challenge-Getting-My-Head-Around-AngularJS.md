title: "Learning 30 Technologies in 30 Days: A Developer Challenge - Getting My Head Around AngularJS"
date: 2016-03-04 18:32:32
tags:
---
今天我打算学习一下AngularJS的基本知识，并希望能用它做一个简单小应用。我也会在这篇文章里用到Bower，我不可能在一天之内学习完AngularJS，所以我打算用好几天时间来学习，每天涉及其中不同的点。
<!--more-->

### 什么是AngularJS？
- 扩展HTML添加动态特性，因此我们可以轻松地构建现代web应用程序
- 以你想要的方式使用
- 带你回到HTML
- 声明方法
- 简单
- 通过双向数据绑定消除DOM操作，任何时候当模型改变时视图都会得到更新，反之亦然
- 你可以用它来构建单页Web应用程序。当你构建如路由，Ajax调用，数据绑定，缓存，历史记录和DOM操作这类的SPA应用时，会有很多的挑战。

#### AngularJS的主要组件是：
- 控制器：视图背后的代码
- 作用域：包含模型数据，粘合控制器和视图
- 模块：定义新的服务，或使用现有的服务、指令、过滤器等，模块可以依赖于另一个模块
- 指令：允许你通过定义自己项目特定的HTML指令来扩展HTML，学习HTML的新花样
![img](http://segmentfault.com/img/bVbDc7)

### 为什么我会在意AngularJS？
对我而言有两个主要原因：
- 它是由谷歌支持，有很多开发者的大社区
- 全栈框架：这意味着我不需要依靠其他数百万计的脚本，它们会很好地整合在一起

### 前提准备
我们将使用Bower为示例应用程序安装AngularJS，如果你还没有安装bower，那么请看我[前一篇文章](http://csrgxtu.github.io/2016/03/04/Learning-30-Technologies-in-30-Days-A-Developer-Challenge-bower/)

### 安装AngularJS
在系统的任何方便的位置创建一个叫 bookshop 的目录，用下面命令来安装AngularJS和Twitter bootstrap：
```bash
$ bower install angular
$ bower install bootstrap
```
上面的命令会在bookshop目录下创建一个叫bower_components的文件夹，里边有已安装的全部组件。

### 开始使用AngularJS
现在创建一个名为 index.html 的html文件，它将会是一个基于AngularJS的网上书店应用。
```html
<!doctype html>
<html>
<head>
    <title>Bookshop - Your Online Bookshop</title>
    <link rel="stylesheet" type="text/css" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>

    <div class="container">
          <h2>Your Online Bookshop</h2>
    </div>

<script type="text/javascript" src="bower_components/angular/angular.min.js"></script>
</body>
</html>
```
如果你在浏览器打开这个文件，你会看到“你的网上书店”正在呈现，但这并不是AngularJS的厉害之处，所以接下来我们看看它真正有趣的地方：
```html
<!doctype html>
<html ng-app>
<head>
    <title>Bookshop - Your Online Bookshop</title>
    <link rel="stylesheet" type="text/css" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>

    <div class="container" ng-init="books=['Effective Java','Year without Pants','Confessions of public speaker','JavaScript Good Parts']">
        <h2>Your Online Bookshop</h2>
        <ul class="unstyled">
            <li ng-repeat="book in books">
                {{book}}
            </li>
        </ul>
    </div>

<script type="text/javascript" src="bower_components/angular/angular.min.js"></script>
</body>
</html>
```

上边这段代码有一些需要注意的点：
- 在HTML标签里边，我们已经定义了ng-app。这将初始化AngularJS应用程序，并告诉AngularJS要在这一部分活跃。所以，它在应用程序里是活跃整个html文件的。
- 我们所使用的第二个Angular指令是ng-init。这将初始化书籍数组中的一个，我们可以将它应用在我们的应用程序中。
- 最后一个有趣的部分，是用于遍历集合中的所有元素的ng-repeat指令。Angular将为每个元素增加 li 元素。所以，如果我们在Web浏览器中打开它，将会看到在一个列表中有四本书。

上边是以字符串数组的形式使用数据，但也可以用存储对象的方式，如下：
```html
<!doctype html>
<html ng-app>
<head>
        <title>Bookshop - Your Online Bookshop</title>
        <link rel="stylesheet" type="text/css" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>

        <div class="container" ng-init="books=[{'name': 'Effective Java', 'author':'Joshua Bloch'},{'name': 'Year without Pants', 'author':'Scott Berkun'},{ 'name':'Confessions of public speaker','author':'Scott Berkun'},{'name':'JavaScript Good Parts','author':'Douglas Crockford'}]">
                <h2>Your Online Bookshop</h2>
                <ul class="unstyled">
                        <li ng-repeat="book in books">
                                <span>{{book.name}} written by {{book.author}}</span>
                        </li>
                </ul>
        </div>

<script type="text/javascript" src="bower_components/angular/angular.min.js"></script>
</body>
</html>
```
在上面的代码中，我们创建了一个书籍数组对象，其中每本书对象都有名字和作者。最后，我们在列表中同时列出作者姓名和书籍名称。

### 使用过滤器
Angular提供了过滤器，这有助于格式化数据。你可以使用过滤器来格式化日期、货币、大小写字符、排列顺序和基于搜索的过滤。下面就是一个教你如何利用过滤器来大写的作者姓名和按书名来排序的例子：

```html
<!doctype html>
<html ng-app>
<head>
    <title>Bookshop - Your Online Bookshop</title>
    <link rel="stylesheet" type="text/css" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>

    <div class="container" ng-init="books=[{'name': 'Effective Java', 'author':'Joshua Bloch'},{'name': 'Year without Pants', 'author':'Scott Berkun'},{ 'name':'Confessions of public speaker','author':'Scott Berkun'},{'name':'JavaScript Good Parts','author':'Douglas Crockford'}]">

        <h2>Your Online Bookshop</h2>
        <ul class="unstyled">

            <li ng-repeat="book in books | orderBy :'name'">
                <span>{{book.name}} written by {{book.author | uppercase}}</span>
            </li>
        </ul>
    </div>

<script type="text/javascript" src="bower_components/angular/angular.min.js"></script>
</body>
</html>
```
正如你所看到的，我们在 ng-repeat 指令中使用了按顺序排列的过滤器，在显示作者姓名时用一个大写过滤器。  

为了添加一个搜索过滤器，添加搜索输入的文本框，然后使用一个过滤器以搜索限制结果，如下：
```html
<!doctype html>
<html ng-app>
<head>
    <title>Bookshop - Your Online Bookshop</title>
    <link rel="stylesheet" type="text/css" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>

    <div class="container" ng-init="books=[{'name': 'Effective Java', 'author':'Joshua Bloch'},{'name': 'Year without Pants', 'author':'Scott Berkun'},{ 'name':'Confessions of public speaker','author':'Scott Berkun'},{'name':'JavaScript Good Parts','author':'Douglas Crockford'}]">

        <h2>Your Online Bookshop</h2>
               <input type="search" ng-model="criteria">
        <ul class="unstyled">

            <li ng-repeat="book in books | filter:criteria | orderBy :'name'">
                <span>{{book.name}} written by {{book.author | uppercase}}</span>
            </li>
        </ul>
    </div>

<script type="text/javascript" src="bower_components/angular/angular.min.js"></script>
</body>
</html>
```

### 使用控制器
控制器是AngularJS的主要组件之一，它们是给视图提供动力和数据的代码。在我们的例子中，我们可以将测试数据初始化代码移到一个控制器，创建一个名为app.js的JavaScript文件，它将容纳我们应用程序所有特定的JavaScript代码。
```javascript
function BookCtrl($scope){
        $scope.books = [
                {'name': 'Effective Java', 'author':'Joshua Bloch'},
                {'name': 'Year without Pants', 'author':'Scott Berkun'},
                { 'name':'Confessions of public speaker','author':'Scott Berkun'},
                {'name':'JavaScript Good Parts','author':'Douglas Crockford'}
        ]
}
```
$scope 控制器和视图之间的粘合剂，而且会注入到BookCtrl。现在我们添加书籍数组的对象到 $scope 对象，这个对象对视图是可见的。

现在改变index.html使用BookCtrl，如下：
```html
<!DOCTYPE html>
<html ng-app>
<head>
    <title>Bookshop - Your Online Bookshop</title>
    <link rel="stylesheet" type="text/css" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>

    <div class="container" ng-controller="BookCtrl">
        <h2>Your Online Bookshop</h2>
        <input type="search" ng-model="criteria">
        <ul class="unstyled">
            <li ng-repeat="book in books | filter:criteria | orderBy :'name'">
                <span>{{book.name}} written by {{book.author | uppercase}}</span>
            </li>
        </ul>
    </div>

<script type="text/javascript" src="bower_components/angular/angular.min.js"></script>
<script type="text/javascript" src="app.js"></script>
</body>
</html>
```
今天就这些内容，这也只是冰山一角。我将会用很多天来学习AngularJS的特性，它真的是一个神奇又强大的库。

中文转载[SegmentFault](https://segmentfault.com/a/1190000000350125)  
英文[Getting My Head Around AngularJS](https://www.openshift.com/blogs/day-2-angularjs-getting-my-head-around-angularjs)
