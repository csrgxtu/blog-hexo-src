---
title: How Python manage memory
date: 2020-02-11 09:28:45
tags: ["Python", "Memory Management"]
---
Python is a garbage collection language, means programmers don't need to care about memory when coding. until we meet high memory usage in production environment and don't know why our Django web application takes so much memory. to understand why high memory usage in Python web application, need to know following:  
* CPython's memory management model
* CPython's GC
* What can we do to reduce memory usage
<!--more-->

### Python implementations
Python have many implementations, like CPython, PyPy, Jython, IronPython etc. each of the implementation have it's own memory management model and Garbage collector. In most case, we use CPython, which is writen in C language. thus i will only cover CPython's memory management model and garbage collector.  


### How old version CPython manage memory
In older version of Python, like version before 2.2, CPython use `glibc`'s `malloc` and `free` allocate and release memory directly. when u create objects, CPython will call `glibc`'s method to allocate memory. when objects no longer needed, it will freeed by `glibc`. without memory management, this method works but not efficiency. cuz we will constantly create new objects and release them, this will cuz constantly invocation of `glibc`, thus slow down the Python code. but, this way is simple, and won't waste memory(only allocate memory when needed and return it back to OS when no need).  

here is the corresponding logic chart for old version CPython  
![old version cpython memory layer](/img/old_version_cpython_mem_logic.png)  

