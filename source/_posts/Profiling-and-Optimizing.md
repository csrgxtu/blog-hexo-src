---
title: Profiling-and-Optimizing
date: 2020-02-01 19:48:12
tags: ['Profiling', 'Optimization', 'Python']
---
大学时看到下面的说法，深以为然，因此一直写代码很少优化。那个时候考虑更多的是如何跑起来和跑正确的问题，很少涉及跑的更快更好的问题。
> 过早的优化是万恶之源 --《计算机程序设计艺术》

如今因为工作关系，我不得不考虑如何跑的更快，跑的更好了。
<!--more-->

### 如何跑的更快呢？
当程序运行正确后，需要考虑如何跑的更快，占用资源更少。其最终的目的是用最少的硬件资源为更多的用户提供服务。这里大致分为两种途径：  
1， 业务调整优化  
2， 代码调整优化  
业务调整优化一般需要调整业务，通过舍弃一些不太重要的功能来大幅提升当前主功能的速度，需要跨team合作。比如当前页面可能不太需要的一些数据（获取比较耗时）可以舍弃。接口或者页面处理的数据少了，速度就会相应的提升。  
代码调整优化则是开发者在不影响当前业务功能的前提下，内部调整优化代码逻辑。整个过程对终端用户是透明的。  

本文我主要说明代码优化，这个是我们开发者在不依赖其它外部资源情况下就可以进行的。

### Profiling
我要优化代码，那该从哪里优化呢？首先当然是定位，定位需要优化的地方。第一种方法是大致评估代码中哪一个模块或者函数比较耗时，然后重点分析。第二种方法是借助工具量化评估代码耗费资源情况（CPU，Mem），根据评估列表结果优化。第一种方法比较主观，因为不同的开发者往往评估的有差距，或者大型项目开发者对其它模块不太熟悉，无法正确评估。而第二种方法比较客观，通过监控统计代码耗费资源的情况量化其性能，开发者可以很容易从统计的结果中得出代码哪一块比较耗费资源。第二种方法就是Profiling。

Profiling 一般使用下面的工具：  
1， Python装饰器，用来测量函数耗时
```Python
import time
from functools import wraps

def timefn(fn):
    @wraps(fn)
    def measure_time(*args, **kwargs):
        t1 = time.time()
        result = fn(*args, **kwargs)
        t2 = time.time()
        print ("@timefn:" + fn.func_name + " took " + str(t2 - t1) + " senconds")
        return result
    return measure_time

@timefn
def func_to_measure():
    pass
```

2， Python Module `timeit`
```Python
python -m timeit -n 5 -r 5 ...

5 loops, best of 5: 13.1 sec per loop
```

3， Unix Time Command
```bash
/usr/bin/time -p python scripts.py
...

real 13.46
user 13.40
sys 0.04
```

4, cProfile module  
可以查看当前程序中用户定义函数或者Python内部函数的调用及耗时情况。一般用来定位函数层级。
```bash
python -m cProfile -s cumulative problem1.py 

         66716 function calls in 9.435 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        3    0.000    0.000    0.000    0.000 functools.py:17(update_wrapper)
        3    0.000    0.000    0.000    0.000 functools.py:39(wraps)
        1    0.000    0.000    9.435    9.435 problem1.py:1(<module>)
        1    9.407    9.407    9.408    9.408 problem1.py:20(merge_two_list)
        1    0.019    0.019    0.025    0.025 problem1.py:4(get_muliply_res)
        1    0.001    0.001    0.001    0.001 timing_decorators.py:1(<module>)
        3    0.000    0.000    0.000    0.000 timing_decorators.py:5(timefn)
        2    0.000    0.000    9.433    4.717 timing_decorators.py:6(measure_time)
       15    0.000    0.000    0.000    0.000 {getattr}
    66665    0.005    0.000    0.005    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.000    0.000    0.000    0.000 {method 'format' of 'str' objects}
        3    0.000    0.000    0.000    0.000 {method 'update' of 'dict' objects}
        2    0.003    0.002    0.003    0.002 {range}
        9    0.000    0.000    0.000    0.000 {setattr}
        1    0.000    0.000    0.000    0.000 {sum}
        4    0.000    0.000    0.000    0.000 {time.time}
```

5, line_profiler module
上述cProfile定位到函数级别后，可以使用line_profiler定位到函数中具体某一行。
```bash
kernprof -l -v problem1.py 


Total time: 9.80845 s
File: problem1.py
Function: merge_two_list at line 20

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    20                                           @timefn
    21                                           @profile
    22                                           def merge_two_list(small_lst, big_lst):
    23                                               """ merge two list into one, remove duplicate elements """
    24         1        158.0    158.0      0.0      res = big_lst
    25     20000       7026.0      0.4      0.1      for ele in small_lst:
    26     19999    9792981.0    489.7     99.8          if ele not in res:
    27     13333       8285.0      0.6      0.1              res.append(ele)
    28
    29         1          0.0      0.0      0.0      return res
```

6, memory_profiler
前面的工具都是测量程序运行耗时的，这个工具用来测量内存占用情况。
```bash
python -m memory_profiler problem1.py

Filename: problem1.py

Line #    Mem usage    Increment   Line Contents
================================================
    20   30.691 MiB   30.691 MiB   @timefn
    21                             @profile
    22                             def merge_two_list(small_lst, big_lst):
    23                                 """ merge two list into one, remove duplicate elements """
    24   30.695 MiB    0.004 MiB       res = big_lst
    25   30.977 MiB    0.000 MiB       for ele in small_lst:
    26   30.977 MiB    0.168 MiB           if ele not in res:
    27   30.977 MiB    0.004 MiB               res.append(ele)
    28
    29   30.977 MiB    0.000 MiB       return res
```

还有其它工具这里将不赘述。

### 实验
从1w个电影title中找出重复的title并打印出来，数据如下：
```bash
Carmencita
Le clown et ses chiens
Pauvre Pierrot
Un bon bock
Blacksmith Scene
Chinese Opium Den
Corbett and Courtney Before the Kinetograph
Edison Kinetoscopic Record of a Sneeze
Miss Jerry
Exiting the Factory
...
```
[titles_1w.txt](/img/titles_1w.txt)

代码如下：
```python
def read_movies(src):
    with open(src) as fd:
        return fd.read().splitlines()


def is_duplicate(needle, haystack):
    for movie in haystack:
        if needle.lower() == movie.lower():
            return True
    return False


def find_duplicate_movies(src="titles_1w.txt"):
    movies = read_movies(src)
    duplicates = []

    while movies:
        movie = movies.pop()
        if movie in movies:
            duplicates.append(movie)

    return duplicates


if __name__ == '__main__':
    res = find_duplicate_movies()
    print res
```

运行上面的脚本发现耗时35秒
```bash
python -m cProfile find_duplicate_movies.py

         97560705 function calls in 35.499 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000   35.499   35.499 find_duplicate_movies.py:1(<module>)
        1    0.000    0.000    0.001    0.001 find_duplicate_movies.py:1(read_movies)
        1    0.013    0.013   35.499   35.499 find_duplicate_movies.py:13(find_duplicate_movies)
    10001   12.619    0.001   35.482    0.004 find_duplicate_movies.py:6(is_duplicate)
      376    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
 97540320   22.863    0.000   22.863    0.000 {method 'lower' of 'str' objects}
    10001    0.004    0.000    0.004    0.000 {method 'pop' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'read' of 'file' objects}
        1    0.001    0.001    0.001    0.001 {method 'splitlines' of 'str' objects}
        1    0.000    0.000    0.000    0.000 {open}
```
分析cProfile的结果，可以看出耗时发生在两个函数调用：
```bash
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
10001   12.619    0.001   35.482    0.004 find_duplicate_movies.py:6(is_duplicate)
97540320   22.863    0.000   22.863    0.000 {method 'lower' of 'str' objects}
```
#### 第一次优化
对应代码中的函数：`is_duplicate` 和 build-in: `lower`，可以看出string的`lower`函数比较耗费时间，花费了**22.863**秒。我们可以在读入movie title后直接转化成lower case，进而防止在`is_duplicate`中2次调用`lower`。改进后的代码如下：
```python
def read_movies(src):
    with open(src) as fd:
        return fd.read().splitlines()


def is_duplicate(needle, haystack):
    for movie in haystack:
        if needle == movie:
            return True
    return False


def find_duplicate_movies(src="titles_1w.txt"):
    movies = read_movies(src)
    movies = [movie.lower() for movie in movies]
    duplicates = []

    while movies:
        movie = movies.pop()
        if is_duplicate(movie, movies):
            duplicates.append(movie)

    return duplicates


if __name__ == '__main__':
    res = find_duplicate_movies()
    print res[0:5]

```
运行改进后的程序，结果如下：
```bash
python -m cProfile find_duplicate_movies.py

         30386 function calls in 1.175 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    1.175    1.175 find_duplicate_movies.py:1(<module>)
        1    0.000    0.000    0.001    0.001 find_duplicate_movies.py:1(read_movies)
        1    0.006    0.006    1.175    1.175 find_duplicate_movies.py:13(find_duplicate_movies)
    10001    1.163    0.000    1.163    0.000 find_duplicate_movies.py:6(is_duplicate)
      376    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    10001    0.003    0.000    0.003    0.000 {method 'lower' of 'str' objects}
    10001    0.001    0.000    0.001    0.000 {method 'pop' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'read' of 'file' objects}
        1    0.001    0.001    0.001    0.001 {method 'splitlines' of 'str' objects}
        1    0.000    0.000    0.000    0.000 {open}
```
可以看出，总耗时下降到**1.175**秒。

#### 第二次优化
从上面profiling结果可以看出主要耗时函数为：`is_duplicate`。这里去掉is_duplicate 函数调用，直接在`find_duplicate_movies`中判断当前movie是否在剩余movies中，如下：
```python
def read_movies(src):
    with open(src) as fd:
        return fd.read().splitlines()


def find_duplicate_movies(src="titles_1w.txt"):
    movies = read_movies(src)
    movies = [movie.lower() for movie in movies]
    duplicates = []

    while movies:
        movie = movies.pop()
        if movie in movies:
            duplicates.append(movie)

    return duplicates


if __name__ == '__main__':
    res = find_duplicate_movies()
    print res[0:5]
```

运行改进后的程序，结果如下：
```bash
python -m cProfile find_duplicate_movies.py

         20385 function calls in 0.463 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.463    0.463 find_duplicate_movies.py:1(<module>)
        1    0.000    0.000    0.001    0.001 find_duplicate_movies.py:1(read_movies)
        1    0.457    0.457    0.463    0.463 find_duplicate_movies.py:13(find_duplicate_movies)
      376    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    10001    0.003    0.000    0.003    0.000 {method 'lower' of 'str' objects}
    10001    0.001    0.000    0.001    0.000 {method 'pop' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'read' of 'file' objects}
        1    0.001    0.001    0.001    0.001 {method 'splitlines' of 'str' objects}
        1    0.000    0.000    0.000    0.000 {open}
```
总耗时下降到**0.463**秒。

#### 第三次优化
上述判断是否重复是通过pop出一个title后，然后检查title是否在剩余的list中。这里我们可以使用更高效的方法，先将list排序，那么相同的title将会在list中紧挨着，这个时候只要循环排序后的数组，比较idx和idx+1的元素即可。代码如下：
```python
def read_movies(src):
    with open(src) as fd:
        return fd.read().splitlines()


def find_duplicate_movies(src="titles_1w.txt"):
    movies = read_movies(src)
    movies = [movie.lower() for movie in movies]
    movies.sort()
    duplicates = []
    for idx in range(len(movies) - 1):
        if movies[idx] == movies[idx + 1]:
            duplicates.append(movies[idx])

    return duplicates


if __name__ == '__main__':
    res = find_duplicate_movies()
    print res[0:5]
```

运行改进后的程序，结果如下：
```bash
python -m cProfile find_duplicate_movies.py

         10387 function calls in 0.010 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.010    0.010 find_duplicate_movies.py:1(<module>)
        1    0.000    0.000    0.001    0.001 find_duplicate_movies.py:1(read_movies)
        1    0.002    0.002    0.010    0.010 find_duplicate_movies.py:13(find_duplicate_movies)
        1    0.000    0.000    0.000    0.000 {len}
      376    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    10001    0.003    0.000    0.003    0.000 {method 'lower' of 'str' objects}
        1    0.000    0.000    0.000    0.000 {method 'read' of 'file' objects}
        1    0.003    0.003    0.003    0.003 {method 'sort' of 'list' objects}
        1    0.001    0.001    0.001    0.001 {method 'splitlines' of 'str' objects}
        1    0.000    0.000    0.000    0.000 {open}
        1    0.000    0.000    0.000    0.000 {range}

```
总耗时下降到**0.010**秒。