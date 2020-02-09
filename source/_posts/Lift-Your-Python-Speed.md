---
title: Lift Your Python Speed
date: 2020-02-09 12:00:54
tags: ["Python", "Optimization"]
---
> The easiest way to get your code to run faster is to make it do less work. assuming you've already chosen good algorithms and you've reduced the amount of data you're processing, the easiest way to execute fewer instructions is to compile your code down to machine code.
<!--more-->

In previous post [Profiling and Optimizating](/2020/02/01/Profiling-and-Optimizing/) I talked about how to profiling and optimizing your code, apart from that, what can we do to make the code run faster?  

Python offers a number of options for this, including pure C-based compiling approaches like Cython, ShedSkin and Pythran; LLVM-based compiling via Numba; and the replacement virtual machine PyPy, which includes a built-in just-in-time(JIT) compiler. you need to balancethe requirements of code adaptability and team velocity when deciding which route to take.   
In this post I will focus on PyPy and Cython.

### What sort of speed gains are possible?
Python code that tends to run faster after compiling is probably mathematical and it probably has lots of loops that repeat the same operations many times. Inside these loops, you're probably making lots of temporary objects.  

Code that calls out to external liraries (e.g regular expressions, string operations, calls to database libraries) is unlikely to show any speed up after compiling. Programs that are I/O-bound are also unlikely to show significant speedups. also if your code call `numpy` or `scipy`, it may not run any faster after compile to `C` code.  


Usually, some effort profiling and compiling brings a lot of reward, but continued effort tends to pay inscreasingly less. same to *100 meters race*, the faster you run, the harder you train.

### JIT versus AOT compilers
There are two kinds of compilers: JIT (just in time) and AOT (ahead of time).   

JIT compilers will step into code and compile it when you using it, you don't need do much work, but it have a cold start problems. normally the second time run will be much faster. PyPy and Numba are the representations of JIT compilers.  

AOT compilers will compile code first into corresponding machine code, then this compiled lib can be used instantly. It needs you change code correspondingly. Cython, ShedSkin, Pythran are the representations of AOT compilers.  

|  AOT   | JIT |
|  ----  | ----  |
| Cython  | Numba |
| ShedSkin  | PyPy |
| Pythran |   |

|     | Python2 | Python3 |
|  ----  | ----  | ------ |
| Cython  | Y | Y |
| ShedSkin  | Y | N |
| Pythran |  Y | Y |
| Numba | N | Y |
| PyPy | Y | Y |

### Why does type information help code run faster?
Python is dynamically typed -- a variable can refer to an object of any type, and any line of code can change the type of the object that is referred to. this makes it difficult for the virtual machine to optimize how the code is executed at the machine code level, as it doesn't know which fundamental datatype will be used for future operations. keeping the code generic mkes it run more slowly.  

Inside Python every fundamental object, like an integer, will be wrapped up in a higher level Python object. the higher level object has extra unctions like `__hash__` to assist with storage and `__str__` for printing.  

Inside a section of code that is CPU bound, it is often the case that tye types of variables dont change. this gives us an opportunity for static compilation and faster code execution.  

For mathmatical operations, we don't need high level functions, even the Cpython's reference counting GC. we can use machine level code directly. thus it will not interact with Python's VM and directly run on your machine, this will save you lots of time. by doing this, we usually declare types or use `C` functions directly in Python code.

### Cython
Cython is a compiler that converts type-annotated Python into a compiled extension module. the type annotations are C-like. this extension can be imported as regular Python module using `import`. Getting started is simple, but it does have a learning curve that must be climbed with each additional elvel of complexity and optimization.  

It actually another programming language that can interact with Python. to make our code run faster, we need to anylize the code, find which part interact with Python, which don't. and then optimize the part which interact with Python. by doing so we can find it easily:
```bash
cython -a find_duplicate_movies.pyx
```
![cython annotation](/img/cython_annotation.png)

** But it need a learning curve!!! **, i will cover it later.

### PyPy
PyPy is an alternative implementation of the Python language that includes a tracing just-in-time compiler; it is compatibale with Python 2.7 and an experimental Python3.2 version is available.  

PyPy uses a different type of garbage collector to CPython, and this can cause some nonobvious behavior changes to your code. Whereas CPython uses reference counting, PyPy uses a modified mark and sweep approach that may clean up unused object much later. Both are correct implementations of the Python specification.  for example, when read file, CPython will close the file handler when it's done due to reference counting. but for PyPy, it will close and flush data later.  

### Experiment
here is the example from [Profiling and Optimizing](/2020/02/01/Profiling-and-Optimizing/), find duplicate movies in a list of movie titles from file. here is the data file which contains 6511101 lines: [titles.txt](/img/titles.txt)

```Python
def read_movies(src):
    with open(src) as fd:
        return fd.read().splitlines()


def find_duplicate_movies(src="titles.txt"):
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

run it with CPython directly, it use 10 seconds:
```bash
time python find_duplicate_movies.py

real	0m10.187s
user	0m9.503s
sys	0m0.635s
```

run it with PyPy, it use 7 seconds, much faster then CPython.
```bash
time pypy find_duplicte_movies.py

real	0m7.340s
user	0m6.456s
sys	0m0.724s
```


### Reference
[High Performance Python -- Chapter 7]()  
[Cython](https://cython.org/#documentation)  
[Cython as a game changer](https://www.youtube.com/watch?v=_1MSX7V28Po)  
[Cython to speed up your Python code](https://www.youtube.com/watch?v=zx0wMxuh-wk)  
[PyPy](https://pypy.org/)  