title: "Introduction to NodeJs"
date: 2015-03-16 13:34:59
tags: [NodeJs, JavaScript]
categories: Programming Language
---
![NodeJs Logo](http://23daps42d4g822ubuj3etiws15ld.wpengine.netdna-cdn.com/wp-content/uploads/2014/02/nodejs-logo-225x180.png)
## Abstract
In this article, I am gonna to illustrate the basics of a programming language, i.e. NodeJs.

NodeJs as a programming language quickly draw the programmer's attention in about these years.

If you think that PHP or Ruby has boosted the development of the Web, then this time is NodeJs.

And even more I think NodeJs can do a lot more than web programming
<!--More-->

## Backgrounds
NodeJs isn't a completely new tech, it is envolved from JavaScript programming language.

### A little about JavaScript
If you know a little history about Browsers, you may know there was a company named NetScape. and JavaScript was invented by this company.

At that time, with the war of the NetScape and the IE, the Java programming language invented by Sun coporation becaming famous among programmers, so the NetScape want to add some logical operations in the browser, they asked a programmer to develop a tech(language) that can run in the browsers and do some logical operations with Sun's Java programming language. the only give the programmer one week. so, the JavaScript borned, but with a lot of shortages, i.e., it is not strictly or a friendly OOP, or the compiler in it is so unefficiency. though it works, but not perfectly, the user experience on it is really worse. thus the third party libraries starting appears like JQuery etc.

In about 2008, Google releases it's own browser Chrome with the redesign of the JavaScript compiler namely V8 engine. V8 is responsible compile the client side JavaScript and run it in browser. it is asynchronously, i.e. event driven based.

Then, another programmer named Ryan Dahl, who is a C/C++ programmer, mainly focus on high performance web servers, starting developing a event-driven, asynchronous programming language implemented in C based on V8 engine. and this new tech use JavaScript as its forehead language and C/C++ as its back-end, and can run on both server and client, this is NodeJs.

## The NodeJs
From the Backgrounds, you should know a little history about NodeJs. in this section, you are gonna to learn the basics of NodeJs.

### Installing
NodeJs is a cross platform programming language, you can easily run it on Linux, Mac and Windows. unlike Python, Perl, PHP, you wont need to install a third party version NodeJs on Windows. For installing the NodeJs, check out [[Installing](https://nodejs.org/download/)].

If you are a Linux user, just download the binary package for your platform(32 or 64 bit), then uncompress the package, and add the bin directory in the package to the PATH.

For Other platforms, please read the docs.

### Development
If you installed the NodeJs on your system, it is time to start developing applications use NodeJs.

To develop NodeJs applications, you don't need to install eclipse or something else like Java doese or Android dose. just a plain text editor, like VIM, Emacs etc. I am starting developing by using Vim, and then move to the [Atom](https://atom.io/) editor, an editor developed by GitHub, an editor for Hackers.

NodeJs use JavaScript syntax, and JavaScript is really simple, if you don't know about JavaScript, take a little time read the [JavaScript Tutorial](http://www.w3schools.com/js/).

NodeJs comes with many packages, just like Python's third party packages, or PHP's or Perl's. you can easily install a package by using command NPM. for example:
```bash
npm install package-name
```
with these packages, you can quickly accomplish some tasks without inventing the wheels by your own.

NodeJs comes with a interactive shell, just like Python's, and you can quickly test some ideas in it, just type node in your shell, and you will see:
```bash
$node
>
```

### Frameworks
If you are developing Web applications, NodeJs comes with bunch of frameworks, from simple to complicated, like [Express.js](http://expressjs.com/), [Sails.js](http://sailsjs.org/) and [Koa.js](http://koajs.com/).

### Unit Test
Sure, NodeJs support unit test, for example, [Unit.js](http://unitjs.com/) and a lot more.

### The Advantage
NodeJs is event driven and asynchronous language, Just like Nginx, asynchronous way is very fast and suitable for high peformance servers.

### The Disadvantage
NodeJs is single threaded, thus if you are a multi core CPU, it will not make fully usage of it, but there are some solutions for it right now. for example [Cluster](https://nodejs.org/api/cluster.html) and so on ...

## Summary
In this post, you should know what NodeJs is. but it isnt a tutorial on How to programming with NodeJs Step by Step, there is a lot such resources on the internet. you can Google yourself.

## Reference
[NodeJs](https://nodejs.org/) - https://nodejs.org/

[NodeJs Wiki](http://en.wikipedia.org/wiki/Node.js) - http://en.wikipedia.org/wiki/Node.js

[JavaScript wiki](http://en.wikipedia.org/wiki/JavaScript) - http://en.wikipedia.org/wiki/JavaScript

[JavaScript Tut](http://www.w3schools.com/js/) - http://www.w3schools.com/js/

[深入浅出NodeJs](http://book.douban.com/subject/25768396/) - 朴灵 - 人民邮电出版社 - http://book.douban.com/subject/25768396/

[Node.js实战](http://book.douban.com/subject/25870705/) - [美] Mike Cantelon - 人民邮电出版社 - http://book.douban.com/subject/25870705/
