---
title: How Python manage memory
date: 2020-02-11 09:28:45
tags: ["Python", "Memory Management"]
---
Python is a garbage collection language, means programmers don't need to care about memory when coding. until we meet high memory usage in production environment and don't know why our Django web application takes so much memory. to understand why high memory usage in Python web application, need to know following:  
* CPython's memory management model
* CPython's GC
* PyPy's memory management model
* PyPy's GC
* What can we do to reduce memory usage

<!--more-->
