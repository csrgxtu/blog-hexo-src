---
title: Speed up your code with concurrency
date: 2020-02-11 09:55:06
tags: ["Python", "Concurrency"]
---
Why my code still slow after i [Profiling and Optimizing](/2020/02/01/Profiling-and-Optimizing/), or even [Lift Your Python Speed](/2020/02/09/Lift-Your-Python-Speed/)? what else could i do to improve the speed of my code? the answer is `concurrency`!
<!--more-->

There are two kinds of tasks, i.e CPU bound or I/O bound task. if your task mainly focus on calculation, then it's a CPU bound, if your task mainly spend time waitting File I/O, Network I/O etc, then its a I/O bound task. when applying concurrency to your code, you need make yourself clear of following:  
* task type
* concurrency types
