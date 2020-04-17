---
title: 'Gevent: patch it and pray'
date: 2020-04-16 17:42:36
tags: ['Python', 'Gevent', 'Patch']
---
To speedup the IO related operations in our project, we use Gevent and patch_all in Django. but it also makes project more complicated, especially when a new maintainer trying to figure out how the multi-threading process works without knowing the patch. So Guido van Rossum once said: when you use gevent in your project, once after patch, god knows what will happen later. i.e Patch it and Pray!!! and also for other reasons, this way isn't a Pythonic way. here in this post, let's see how it works, why it is bad.
<!--more-->
### What's monkey patch
In Python, monkey patch is simply the dynamic replacement of attributes at runtime. for instance, consider a class that has a method `get_data`. this method does an external lookup(on a database or web api, for example), and vrious other methods in the class call it. However, in a unit test, you don't want to depend on the external data source - so you dynamically replace the `get_data` method with a stub that returns fixed data.  

Becuase Python classes are mutable, and methods are just attributes of the class, you can do this as much as you like - and, in fact, you can even replace classes and functions in a module in exactly the same way. here is the example of Patch in Python, when in test/dev env, it will use `DataManagerPatcher.get_data` which will return mock data.

```Python
import requests

class DataManager(object):
    @classmethod
    def get_data(cls):
        """
        get data from http api
        :return: str
        """
        resp = requests.get('https://google.com')
        return resp.content

class DataManagerPatcher(object):
    @classmethod
    def get_data(cls):
        """
        get data mock
        :return: str
        """
        return '<html></html>'

    @classmethod
    def patch(cls):
        """
        path the get_data with mock method
        """
        DataManager.get_data = cls.get_data


if __name__ == '__main__':
    # if test/dev env, need use mock data
    if os.get('env') in ['DEV', 'TEST']:
        DataManagerPatcher.patch()

    DataManager.get_data()
```

But why monkey patch can work in Python like this? it is NameSpace. In python both attributes and methods are called attributes, they have name bind to object. and Python use namespaces to implement this scoping. it is a dict like data structure with each name as key and object as value. when you access an attribute, Python VM will lookup the namespace and call the corresponding object. if no name found, you got a `NameError` exception. also it following LEGB rules, which is :

![LEGB](https://blog.confirm.ch/wp-content/uploads/2017/08/python_namespaces_legb.jpg)  

So, patch is achived by replace the name-object binding in current process's namespace. but use caution when monkey patching due to following reasons:  
* if anything else besides your test logic calls `get_data` as well, it will also call your monkey-patched replacement rather than the original -- which can be good or bad, just beware.
* if some variable or attribute exists that also points to the `get_data` function by the time you replace it, this alias will not change its meaning and will continue to point to the original `get_data`. (Python already have that binding)

### How Gevent work
Gevent is a co-routine based Python networking library that uses greenlet to provide a high level synchronous API on top of the libev or libuv event loop which implements asynchronous I/O model. basicly, it use eventloop schedule co-routines, and co-routine will replase control when entring I/O to event loop. for more detail, here i recommend
Kavya's [A deep dive into how gevent works](https://www.youtube.com/watch?v=GunMToxbE0E&t=1536s).  

And with monkey-patching, gevent can replace the corresponding 3rd party lib method. your request eventually gose to Gevent's corresonding method.

```Python
from gevent import monkey; monkey.patch_all()
import gevent
import urllib2

def f(url):
    print('GET: %s' % url)
    resp = urllib2.urlopen(url)
    data = resp.read()
    print('%d bytes received from %s.' % (len(data), url))

gevent.joinall([
        gevent.spawn(f, 'https://www.python.org/'),
        gevent.spawn(f, 'https://www.yahoo.com/'),
        gevent.spawn(f, 'https://github.com/'),
])
```

### The bad
Why it is bad? for following reasons:
* Python3 already support asynchronous I/O model, you should use it instead of 3rd party lib: Gevent
* Gevent only works on CPython, not support other Python intepreter like PyPy or Jthon...
* Dynamic replace change the binding between name-object is a bad idea, it will confuse other people
it validates _The Zen of Python_:
* Explicit is better than implicit
* If the implementation is hard to explain, it's a bad idea.

### Reference
[A deep dive into how gevent works](https://www.youtube.com/watch?v=GunMToxbE0E&t=1536s)  
[Monkey patch 解惑](https://zpzhou.com/archives/monkey_patch.html)  
[Python namespaces](https://blog.confirm.ch/python-namespaces/)  
[gevent.org](http://www.gevent.org/)  
