---
title: CPython's Garbage Collector
date: 2020-02-18 10:56:50
tags: ['Python', 'CPtyhon', 'GC', 'Memory Management']
---
As described in [How Python manage memory](/2020/02/11/How-Python-manage-memory/), we now know how CPython allocate memory for objects, but how CPython reclaim that memory? this post will show you how CPython reclaim memory through Garbage Collector.
<!--more-->

### Reference Count
CPython use reference count relaim memory, we know that everything in Python is object, `int`, `str`, `list` or `dict` etc, which all derived from `PyObject`, which have following struct defination in C:
```c
typedef struct _object {
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;
```
the `ob_type` field indicated current object type, i.e `int` or `str` etc. the `ob_refcnt` is a number, it will be incremented by 1 in following situations:
* variable assignment
* function argument passing
* appending to a container object

In Python we can get this reference count easily with `sys.getrefcount`, here is an example:
```Python
import sys


foo = []  # now list object's ref count is 1
print sys.getrefcount(foo)  # will print 2, cuz getrefcount function will increase 1 count to ref count
```
when an object's reference count goes to zero, it will be automatically freed by Python to it's memory pool. and Python will return memory to operating system only and only if the whole `arena` memory area are empty. but due to **memory fragement**, all `arenas` will in used situation, thus Python will hold lots of memory when it is running.

Reference count is incredibly simple and efficient, but it cant deal with reference cycles. thus Python have an supplemental algorithm called _generation garbage collection_, .

The reference counting module is fundamental to Python thus cant be disabled, but _generation garbage collection_ is optional and can be disabled manually like following:

```Python
import gc


gc.disable()
...
```

### Reference Cycles
A reference cycle occurs when one or more objects referencing each other or itself. 

![objects referencing each other](https://rushter.com/static/uploads/img/circularref.svg)

For container objejcts like `list`, `tuppel`, `dict`, `class` etc, will meet this cycle.

here is an example:
```Python
import gc
import sys
import ctypes


# disable generation gc first
gc.disable()


# objects referencing each other
object_one = {}  # refcnt: 1
object_two = {}  # refcnt: 2

object_one_addr = id(object_one)
object_two_addr = id(object_two)

object_one['object_two'] = object_two  # refcnt: 2
object_two['object_one'] = object_one  # refcnt: 2

del object_one, object_two  # after del, the object are unreachable, but the refcnt is still 1, thus cant be reclamed
print ctypes.c_long.from_address(object_one_addr).value  # 1
print ctypes.c_long.from_address(object_two_addr).value  # 1

# object referencing itself
lst_obj = []  # refcnt: 1
lst_obj_addr = id(lst_obj)
lst_obj.append(lst_obj)  # refcnt: 2
del lst_obj  # refcnt: 1, after del, refcnt to object is still 1, thus cant be cleaned
print ctypes.c_long.from_address(lst_obj_addr).value  # 1
```
We know that Reference Counting only free object when it's `ob_refcnt` field is zero. but objects which have reference cycles never goes to zero, thus Reference Counting can't free it.

### Generational Garbage Collection
To solve reference cycle problems, CPython use Generational Garbage Collection. Generation GC will run periodically(whenever the number of objects gets over the threshhold `gc.get_threshhold()`).

#### How to detect reference cycles
CPython only focus on container objects, i.e. objects that can contain another object: arrays, dictrionaries, user classes etc. also GC will ignore tupples with only immutable types(int, str).   

For this CPython have two doubly linked list: one of list objects to be scanned, and a tentatively unreachable list.  

Let's take an example: 
```Python
# object self referencing
lst_obj = []
lst_obj.append(lst_obj)

# objects ref each other
object_one = {}
object_two = {}
object_three = {}

A = object_one
object_one['object_two'] = object_two
object_two['object_three'] = object_three
object_three['object_one'] = object_one
```

1, when gc starts, it has all objects in first list, each object also has an `gc_ref` field initially equal to `ob_refcnt`, as following image shows, you can ref code `[update_refs](https://hg.python.org/cpython/file/eafe4007c999/Modules/gcmodule.c#l335)` for more detailed info:

![doubly linked list](https://pythoninternal.files.wordpress.com/2014/07/python-cyclic-gc-1-new-page.png)

2, GC then iterate each objects in first list. for each object, it will decrements by 1 the gc_ref of the objects that current object refering.

![scan objects](https://pythoninternal.files.wordpress.com/2014/07/python-cyclic-gc-2-new-page.png)

3, GC scan again the first list, objects with `gc_ref` zero are marked as `GC_INTENTATIVE_UNREACHABLE` and moved to intentative unreachable list, as following image shows:

![unreachable intentative list](https://pythoninternal.files.wordpress.com/2014/07/python-cyclic-gc-3-new-page.png)

4, GC scan first list again, cuz link1's gc_ref is 1, it is marked as `GC_REACHABLE`.

![mark reachable](https://pythoninternal.files.wordpress.com/2014/07/python-cyclic-gc-4-new-page.png)  

5, When GC encounters rechable objects, it traverse all objects it referencing, mark them `GC_REACHABEL` too and move them into first list.
![move back](https://pythoninternal.files.wordpress.com/2014/07/python-cyclic-gc-5-new-page.png)  

6, When all objects are scaned, the container objects in tentively unreachable list can be garbage collected.

#### Optimization #1: limiting the time for each collection
The most objects die yong, for example, most objects we created will not be used later, thus can be collected shortly, these are yong objects. on the other side, the older objects are unlikely collected .  

CPython defined 3 generations of objects from 0 to 2. every new object starts at generation 0, when it survives one GC round, it moves to next generation. the older generation, the longer it lives, thus unlikely collected. 

### Summary
CPython use reference counting mainly, will use generational garbage collection as an suplenmental method. to make your code efficient, plz dont write cyclic reference, this will cost time when generational garbage collector runs.

### Reference Resource
[Portable Garbage Collection](http://arctrix.com/nas/python/gc/)  
[PyCon 2019: Time to take out rubbish: garbage collector by Pablo Galindo Salgado](https://www.youtube.com/watch?v=CLW5Lyc1FN8&t=1139s)  
[PyCon 2016: Memory management in Python by Nina Zakharenko](https://www.youtube.com/watch?v=F6u5rhUQ6dU)  
[The Garbage Collector by Lpoulain](https://pythoninternal.wordpress.com/2014/08/04/the-garbage-collector/)  
[Garbage collection in Python: things you need to know by Aertem Golubin](https://rushter.com/blog/python-garbage-collector/)  

