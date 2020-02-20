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
In older version Python, like versions before 2.2, CPython use `glibc`'s `malloc` and `free` allocate and release memory directly. when u create objects, CPython will call `glibc`'s method to allocate memory. when objects no longer needed, it will freeed by `glibc`. without memory management, this method works but not efficiency. cuz we will constantly create new objects and release them, this will cuz constantly invocation of `glibc`, thus slow down the Python code. but, this way is simple, and won't waste memory(only allocate memory when needed and return it back to OS when no need).  

here is the corresponding logic chart for old version CPython  
![old version cpython memory layer](/img/old_version_cpython_mem_logic.png)  

### How CPython manage memory now?
Request memory when needed works fine, but it needs constantly invocation of `glibc` methods. like working with threading or processing, we usaully use `pool`. when need new thread/processing to run, we just create it from pool in our code directly instead of calling `multithreading` or `multiprocessing` directly, this sames us time.  

To make memory allocation or release more efficiently, Python starts using a new memory model since 2.4. here is the logic chart for new memory model:

![py memory model](/img/py-mem-model.png)

CPython has an object allocator that is responsible for allocating memory within the object memory area. This object allocator is where most of the magic happens. It gets called every time a new object needs space allocated or deleted.  

To manage memeory, CPython object allocator use **Arenas**, **Pools**, **Blocks** internally. lets explain them one by one.

#### Arenas
The arena is a chunk of 256kB memory allocated on the heap, which provides memory for 64 pools. here is the logic chart of arena:

![arena png](/img/arena.png)

the structure of the Arena objects looks like following:
```C
struct arena_object {
    uintptr_t address;
    block* pool_address;
    uint nfreepools;
    uint ntotalpools;
    struct pool_header* freepools;
    struct arena_object* nextarena;
    struct arena_object* prevarena;
};
```
All arenas are linked using doubly linked list (the `nextarena` and `prevarena` fields), `address` is the header address of the doubly linked list, it helps to manage them. The `ntotalpools` and `nfreepools` are storing information about currently available pools.  

The freepools field points to the linked list of available pools.  


#### Pools
A collection of blocks of the same size is called a pool. it's 4kb, same with OS's default page size. limiting pool with a fixed size of blocks helps with fragmentation.  

the structure of the Pool objects looks like following:
```C
/* Pool for small blocks. */
struct pool_header {
    union { block *_padding;
            uint count; } ref;          /* number of allocated blocks    */
    block *freeblock;                   /* pool's free list head         */
    struct pool_header *nextpool;       /* next pool of this size class  */
    struct pool_header *prevpool;       /* previous pool       ""        */
    uint arenaindex;                    /* index into arenas of base adr */
    uint szidx;                         /* block size class index        */
    uint nextoffset;                    /* bytes to virgin block         */
    uint maxnextoffset;                 /* largest valid nextoffset      */
};
```

Pools of the same sized blocks are linked together using doubly linked list (the nextpool and prevpool fields). The szidx field keeps the size class index, whereas ref.count keeps the number of used blocks. The arenaindex stores the number of an arena in which Pool was created. 

The freeblock field is described as follows:
```bash
Blocks within pools are again carved out as needed.  pool->freeblock points to
the start of a singly-linked list of free blocks within the pool.  When a
block is freed, it's inserted at the front of its pool's freeblock list.  Note
that the available blocks in a pool are *not* linked all together when a pool
is initialized.  Instead only "the first two" (lowest addresses) blocks are
set up, returning the first such block, and setting pool->freeblock to a
one-block list holding the second such block.  This is consistent with that
pymalloc strives at all levels (arena, pool, and block) never to touch a piece
of memory until it's actually needed.

So long as a pool is in the used state, we're certain there *is* a block
available for allocating, and pool->freeblock is not NULL.  If pool->freeblock
points to the end of the free list before we've carved the entire pool into
blocks, that means we simply haven't yet gotten to one of the higher-address
blocks.  The offset from the pool_header to the start of "the next" virgin
block is stored in the pool_header nextoffset member, and the largest value
of nextoffset that makes sense is stored in the maxnextoffset member when a
pool is initialized.  All the blocks in a pool have been passed out at least
once when and only when nextoffset > maxnextoffset.
```
here is the block size index table:  

|Request in Bytes|Size of allocated block|Size class index|
|:-:|:-:|:-:|
| 1-8  | 8  | 0  |
| 9-16  |  16 | 1  |
| ...  |   |   |
| 505-512  |  512 |  63 |

Pool have three states:  
* Used, partially used, neither full or empty
* Full, all pool's blocks are currently allocated
* Empty, all pool's blocks are currently available for allocation

In order to effiently manage pools, Cpython use an array called `usedpools`, it stores pointers to pools grouped by class as following image shows:

![usedpool array](https://rushter.com/static/uploads/img/usedpools.svg)

Note that pools and blocks are not allocating memory directly, instead, they are using already allocated space from arenas.

#### Blocks
Blocks are a chunk of memory of certain size. there are 64 kinds of blocks, as block size index table shows, your `int` or `list` will finnaly use these blocks.:

|Request in Bytes|Size of allocated block|Size class index|
|:-:|:-:|:-:|
| 1-8  | 8  | 0  |
| 9-16  |  16 | 1  |
| ...  |   |   |
| 505-512  |  512 |  63 |

### Garbage Collector
When you create objects in your code, CPython's object allocator will allocate blocks of memeory from pool or arena(CPython's memory manager). when you leave your code, CPython's garbage collector will free the memory to Python memeory manager.  

For detail, ref post [CPython's Garbage Collector](/2020/02/18/CPython-s-Garbage-Collector/)


### Reference
[Github: CPython's obmalloc.c](https://github.com/python/cpython/blob/7d6ddb96b34b94c1cbdf95baa94492c48426404e/Objects/obmalloc.c)  
[Real Python: Python memory management](https://realpython.com/python-memory-management/)  
[Ruster: Python memory management](https://realpython.com/python-memory-management/)  
[Evan Johes: Improving Python's memory allocator](https://www.evanjones.ca/memoryallocator/)  
[EuroPython 2011: Understading Python's memory model: Mutability and Methods](https://www.youtube.com/watch?v=HHFCFJSPWrI)  

