---
title: How coroutine works internally [Drafting]
date: 2020-05-07 10:47:51
tags: ['coroutine', 'Python']
---
Coroutines are generalization of subroutines. they are used for cooperative multitasking where a process voluntarily yield(give away) control periodically or when idle in order to enable multiple applications to be run simultaneously. compare to thread, coroutine is context-swtich in User space by programming language or by programmer itself, but thread is managed by OS. but anyway, do you know how Python implements coroutine? or how can it yield control away and lately resume back?

### A simple usage example


### The Internal

### References
[coroutine in python](https://www.geeksforgeeks.org/coroutine-in-python/)  
