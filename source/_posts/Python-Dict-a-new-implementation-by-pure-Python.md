---
title: 'Python Dict: a new implementation by pure Python'
date: 2020-04-21 14:57:23
tags: ['Python', 'Dict']
---
Python have builtin implementation for `dict`, which is used to store key-value and also provided other related operations. Due to it is an frequenctly used basic data type, here I will use pure Python implement an `dict`.
<!--more-->

### Hash map
As we learned from data structure, hash map is a data structure that can be used to store key-value and later retrieve value by key from it with almost constant O(1) time.

A hash map use a `hash` function to compute an index, also called hash code into an array of buckets or slots from which the desired value can be found.  

Ideally the hash function will assign each key to a unique bucket, but most hash table use an imperfect hash function, which might cuz hash collisions where the hash function will generates the same index for different keys.  

In a well-dimensioned hash map, the average cost for each look up is constant and also allow arbitrary insertions and deletions of key-value pairs in constant time.

Basically, it will have following complexity:

|   | Avg  | Worst  |
|:-:|---|---|
| Space   | O(n)  | O(n)  |
| Search  | O(1)  | O(n)  |
| Insert  | O(1)  | O(n)  |
| Delete  | O(1)  | O(n)  |

To summary, it will have an hash function, also need an storage. when hash collisions, need to solve the collisions somehow. for more detail, read the reference.

### Basic version
So, basicly we need solve three things:
* a hash function that will take key as input and output an index
* a storage which will be used to hold the key-value pairs
* how to solve hash collision

for first prolem, we can use Python's built-in method `hash` which will take input and output the index for us.
```bash
hash(...)
    hash(object) -> integer

    Return a hash value for the object.  Two objects with the same value have
    the same hash value.  The reverse is not necessarily true, but likely.
```

for second problem, we use Python's `list` as the storage, we store the `[key, value]` into this list. we will hash key and get an index, use this index as the `list` index.

for third problem, if multiple key hashed into same index, we use a sub_list in `storage[index]` to chain multiple key-value pairs into that sub_list. here is a image which shows how to use chainning solve the collision:
![chainning collision solver](https://media.geeksforgeeks.org/wp-content/cdn-uploads/gq/2015/07/hashChaining1.png) 

To set key-value and get value by key like defeault dict `data_map['name'] = 'shopee'` `data['name']`, we need implement magic methods `__setitem__` and `__getitem__`, here is the basic implementation:

```Python
# -*- coding: utf8 -*-
"""
my implementation for dict
"""

class Dictionary(object):
    """
    implementation for dict
    """
    def __init__(self, size=1000):
        """
        use list as storage, each element is also a list which is used
        to solve hash conflict
        """
        self.storage = [[] for _ in range(size)]
        self.size = size
        self.length = 0

    def __setitem__(self, key, value):
        """
        set key value, if conflict, append to the sub list
        """
        storage_idx = hash(key) % self.size
        for ele in self.storage[storage_idx]:
            if key == ele[0]:  # already exist, update it
                ele[1] = value
                break
        else:
            self.storage[storage_idx].append([key, value])
            self.length += 1
    
    def __getitem__(self, key):
        """
        get by key, if not found, raise key error
        :raise: KeyError
        :return: value
        """
        storage_idx = hash(key) % self.size
        for ele in self.storage[storage_idx]:
            if ele[0] == key:
                return ele[1]

        raise KeyError('Key {} dont exist'.format(key))
```
Here use ipython and test our `Dictionary`
```bash
$ ipython
...

In [1]: from dictionary import Dictionary

In [2]: d = Dictionary()

In [3]: d['name'] = 'shopee'

In [4]: d['age'] = 22

In [5]: d['name']
Out[5]: 'shopee'
```
It is workable now, but not perfect yet, like we isnt implement other operations related to `dict` yet.

### More magic methods
The default dict also supports following:
* check if key in current dict
* iterate over keys
* iterate over values
* iterate over key-value pairs
* delete item by key
* get length of current dict
* string representation of current dict
to implement upper functionalities, we can use Python's magic methods, for example, for iterator related functionalities, we can use `__iter__`, for string representation functionality, we can use `__str__` or `__repr__`. for checking the key in current dict, we can use `__contains__`. here is the detailed implementation:

```Python
# -*- coding: utf8 -*-
"""
my implementation for dict
"""

class Dictionary(object):
    """
    implementation for dict
    """
    ...

    def __delitem__(self, key):
        """
        delete key value from current dictionary instance
        :param key: str
        :return: None
        """
        storage_idx = hash(key) % self.size
        for sub_lst in self.storage[storage_idx]:
            if key == sub_lst[0]:
                self.storage[storage_idx].remove(sub_lst)
                self.length -= 1
                return

        raise KeyError('Key {} dont exist'.format(key))

    def __contains__(self, key):
        """
        check if key exist in current diction
        :param key: str
        :return: boolean
        """
        storage_idx = hash(key) % self.size
        for item in self.storage[storage_idx]:
            if item[0] == key:
                return True
        return False

    def __len__(self):
        """
        return length
        :return: int
        """
        return self.length

    def __iterate_kv(self):
        """
        return an iterator
        :return: generator
        """
        for sub_lst in self.storage:
            if not sub_lst:
                continue
            for item in sub_lst:
                yield item

    def __iter__(self):
        """
        return an iterator
        :return: generator
        """
        for key_var in self.__iterate_kv():
            yield key_var[0]

    def keys(self):
        """
        get all keys as list
        :return: list
        """
        return self.__iter__()

    def values(self):
        """
        get all values as list
        :return: list
        """
        for key_var in self.__iterate_kv():
            yield key_var[1]

    def items(self):
        """
        get all k:v as list
        :return: list
        """
        return self.__iterate_kv()

    def get(self, key):
        """
        get value by key
        :param key: str
        :return: value
        """
        try:
            return self.__getitem__(key)
        except KeyError:
            return None

    def __str__(self):
        """
        str representation of the dictionary
        :return: string
        """
        res = []
        for ele in self.storage:
            for key_value in ele:
                if isinstance(key_value[0], str):
                    key_str = '\'{}\''.format(key_value[0])
                else:
                    key_str = '{}'.format(key_value[0])
                if isinstance(key_value[1], str):
                    value_str = '\'{}\''.format(key_value[1])
                else:
                    value_str = '{}'.format(key_value[1])

                res.append('{}: {}'.format(key_str, value_str))
        key_value_pairs_str = '{}'.format(', '.join(res))
        return '{' + key_value_pairs_str + '}'

    def __repr__(self):
        """
        string representation of the class instances
        :return: string
        """
        return self.__str__()

```
To test it, we try to iterate the dict and check if it contains the key:
```bash
...
In [6]: for key in d.keys():
   ...:     print '{} : {}'.format(key, d.get(key))
   ...:
name : shopee
age : 22

In [7]: d
Out[7]: {'name': 'shopee', 'age': 22}

In [8]: 'name' in d
Out[8]: True
```

### Summary
So, this is just a simple implementation of Python dict by using Python's list and hash built-ins. to extend, you can also implement `clean`, `update` etc. i am also curious how to replace the default `dict` in CPython with my own implementation?

### Reference
[Hash table](https://en.wikipedia.org/wiki/Hash_table)  
[Python dictionary implementation](https://www.laurentluce.com/posts/python-dictionary-implementation/)  
[CPython dictobject.c](https://github.com/python/cpython/blob/master/Objects/dictobject.c)  
[Python magic methods](https://old-panda.com/2018/12/16/python-magic-methods/)  
