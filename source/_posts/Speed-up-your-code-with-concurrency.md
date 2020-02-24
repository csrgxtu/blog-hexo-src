---
title: Speed up your code with concurrency
date: 2020-02-11 09:55:06
tags: ["Python", "Concurrency"]
---
Why my code still slow after i [Profiling and Optimizing](/2020/02/01/Profiling-and-Optimizing/), or even [Lift Your Python Speed](/2020/02/09/Lift-Your-Python-Speed/)? what else could i do to improve the speed of my code? the answer is `concurrency`!
<!--more-->

There are two kinds of tasks, i.e CPU bound or I/O bound task. if your task mainly focus on calculation, then it's a CPU bound, if your task mainly spend time waitting File I/O, Network I/O etc, then its a I/O bound task. 

Python have `multi-threading`, `multi-processing`, `asyncio`, for different kinds of task, you need use different kinds of concurrency.

> note concurrency is time sharing, even though multiple tasks `runing` the same time, there are only one task runing in the CPU at a time. but parallism will use multi cpus run the tasks at the same time.

I will use an I/O bound task and CPU bound task show you how these methods can speedup your code.

### Speedup I/O bound task
Task: download page from many sites

#### Sync version
```Python
import requests
import time


def download_site(url, session):
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with requests.Session() as session:
        for url in sites:
            download_site(url, session)


if __name__ == "__main__":
    sites = [
        "http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/",
        "http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```
run it on my mac (Processor: 2.7 GHz Intel Core i7, Memory: 16GB 2133 MHz LPDDR3)
```bash
$ python3 download_sites.py

...
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Downloaded 160 in 147.5717649459839 seconds
```
it took 147.57 seconds. if you run upper code, the time will be different, cuz it depends on your network connection.

#### Multi-threading version
```Python
import concurrent.futures
import requests
import threading
import time


thread_local = threading.local()


def get_session():
    if not hasattr(thread_local, "session"):
        thread_local.session = requests.Session()
    return thread_local.session


def download_site(url):
    session = get_session()
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(download_site, sites)


if __name__ == "__main__":
    sites = [
        "http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/",
        "http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```

```bash
$ python3 download_sites.py

Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Downloaded 160 in 34.54466891288757 seconds
```
it took 34.54 seconds.

#### Multi-processing version

```Python
import concurrent.futures
import requests
import time


def download_site(url):
    session = requests.Session()
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with concurrent.futures.ProcessPoolExecutor(max_workers=5) as executor:
        executor.map(download_site, sites)


if __name__ == "__main__":
    sites = [
        "http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/",
        "http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```

```bash
$ python3 download_sites.py

...
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 32524 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 18547 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Downloaded 160 in 41.209649085998535 seconds
```
it took 41.20 seconds

#### Async-IO version

```Python
import asyncio
import time
import aiohttp


async def download_site(session, url):
    async with session.get(url) as response:
        print("Read {0} from {1}".format(response.content_length, url))


async def download_all_sites(sites):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in sites:
            task = asyncio.ensure_future(download_site(session, url))
            tasks.append(task)
        await asyncio.gather(*tasks, return_exceptions=True)


if __name__ == "__main__":
    sites = [
        "http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/",
        "http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/",
    ] * 80
    start_time = time.time()
    asyncio.get_event_loop().run_until_complete(download_all_sites(sites))
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} sites in {duration} seconds")
```

```bash
$ python3 download_sites.py

...
Read 5112 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 5112 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 5112 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 8104 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Downloaded 160 sites in 14.110821962356567 seconds
```
it took 14.11 seconds.

#### Multi-processing + Async-I/O
```Python
import asyncio
import time
import aiohttp
import concurrent.futures


async def download_site(session, url):
    async with session.get(url) as response:
        print("Read {0} from {1}".format(response.content_length, url))


async def download_all_sites(sites):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in sites:
            task = asyncio.ensure_future(download_site(session, url))
            tasks.append(task)
        await asyncio.gather(*tasks, return_exceptions=True)

def asyc_tasks(sites):
    asyncio.get_event_loop().run_until_complete(download_all_sites(sites))


def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]

if __name__ == "__main__":
    sites = [
        "http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/",
        "http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/",
    ] * 80
    start_time = time.time()

    with concurrent.futures.ProcessPoolExecutor(max_workers=5) as executor:
        futures = [executor.submit(asyc_tasks, s) for s in chunks(sites, 5)]
        for future in concurrent.futures.as_completed(futures):
            pass

    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} sites in {duration} seconds")
```

```bash
$ python download_sites.py

...
Read 8104 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 5112 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Read 8104 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 8104 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 8104 from http://csrgxtu.github.io/2020/02/18/CPython-s-Garbage-Collector/
Read 5112 from http://csrgxtu.github.io/2020/02/11/Speed-up-your-code-with-concurrency/
Downloaded 160 sites in 6.9041831493377686 seconds
```
it took 6.90 seconds.

Here is the summary result of the upper experiments:  

| x  | Sync  | Threading  | Processing  | Async-I/O  | Processing+Async-I/O  |
|:-:|---|---|---|---|---|
| Time(Seconds)  | 147.57  |  35.54 |  41.20 | 14.11  | 6.90  |

For sync version, it request the site one by one, thus most time spend on waiting network I/O. here is a diagram：

![sync version](https://files.realpython.com/media/IOBound.4810a888b457.png)  

By using Threading or Processing, OS will switch the threading or processing, when one task is waitting or running, OS will put other task into running, thus will reduce the total time on waitting I/O, here is the diagram for threading.:

![threading](https://files.realpython.com/media/Threading.3eef48da829e.png)  

here is the multi-processing diagram:

![processing](https://files.realpython.com/media/MProc.7cf3be371bbc.png)  

As we can see, multi-processing version is slower than multi-threading version, one reason is that process is heavier than thread, it needs to start new interpreter for each process.

Both multi-threading and multi-processing depend on OS's process management. i.e we don't know when our task will be switched from running to pending.   

Async-I/O will use event loop, you need decide when your task need to switch. i.e if it comes to I/O, you need switch to another task, when I/O done, you need process it. if you forget it when coding, it will pending the main loop forever!!! here is the diagram:

![async](https://files.realpython.com/media/Asyncio.31182d3731cf.png)  

For more information on Async, plz refer [async](https://stackoverflow.com/questions/49005651/how-does-asyncio-actually-work/51116910#51116910) or google `how async works?`

Note: Python 2 dont have async support yet, but you can use 3rd party libs: `gevent` or `tonardo`. plz use latest Python version.

### Speedup CPU bound task
Task: computes the sum of the squares of each number from 0 to the passed-in value

#### sync
```Python
import time


def cpu_bound(number):
    return sum(i * i for i in range(number))


def find_sums(numbers):
    for number in numbers:
        cpu_bound(number)


if __name__ == "__main__":
    numbers = [5000000 + x for x in range(20)]

    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print(f"Duration {duration} seconds")
```

```bash
$ python3 cpu_bound.py

Duration 11.239729881286621 seconds
```
this will be done on a signle thread on a signle process on a signle cpu. here is the diagram:

![sync cpu bound](https://files.realpython.com/media/CPUBound.d2d32cb2626c.png)  

#### theading or async version
if you rewrite upper code into threading or async, it will slow down, cuz threading context switch or code complicity added by both will take additional time.


#### multiprocessing
Python's `multiprocessing` module is designed to share heavy CPU workloads to multi CPUS.

```Python
import time
import multiprocessing


def cpu_bound(number):
    return sum(i * i for i in range(number))


def find_sums(numbers):
    with multiprocessing.Pool() as pool:
        pool.map(cpu_bound, numbers)


if __name__ == "__main__":
    numbers = [5_000_000 + x for x in range(20)]

    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print(f"Duration {duration} seconds")
```

```bash
$ python3 cpu_bound.py
Duration 3.2296340465545654 seconds
```

as you can see, it only use 3.22 seconds with multi-processing.

here is the diagram:

![multi-cpus](https://files.realpython.com/media/CPUMP.69c1a7fad9c4.png)



### Conclusion
For tasks that wont run frequently, it is no need to use concurrency, cuz adding concurrency support need additional work to change your code and also increases the complecity. if your code is I/O bound, then `Async-I/O` or `Multi-Processing + Async-I/O` will be a better choice. for CPU bound tasks, use `Multi-Processing`.  

_My Django app is slow? think again!!!_

### Rerence
[how async works](https://stackoverflow.com/questions/49005651/how-does-asyncio-actually-work/51116910#51116910)  
[Python concurrency from ground up by David Beazley from Pycon 2015](https://www.youtube.com/watch?v=MCs5OvhV9S4)  
[Python concurrency by Jim Anderson](https://realpython.com/python-concurrency/)  
