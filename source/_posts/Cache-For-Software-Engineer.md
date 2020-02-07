title: Cache For Software Engineer
date: 2020-01-17 15:00:00
tags: [Cache, Backend, Software Engineer, Devlopper]
categories: [Cache, Software Engineer]
---
In this post i am gonna talk about cache, specifically on application level, will not cover the internals of cache such as Redis/Memcache implementation etc. will cover following topics:  
* What is cache
* Usage senarios
* Cache Types
* Cache Patterns
* 3 major problems and solutions in cache world
<!--more-->

### What is cache
> cache is data redundancy, to achieve higher data access speed.

normally we use memory in current process or use Redis/Memcache to store data instead of directly access DB or other data source to retrieve data. as we all know that memory is much faster than disk or network IO. so use memory as a cache storage is a reasonable choice.

### Usage senarios
what's good senarios with cahce and bad with cache?

Good  
1, low real-time requirements data  
2, high concurrency, like home page for ecomerce website  
3, hot zone data, like most viewed products  
4, low consitency requirements  

Bad  
1, high consistency requirements, like wallet withdraw  
2, rarelly accessed data  
3, data size too big, like the data is about 50M  
4, admin api

### Cache Types
To summarize, there are following cache types:

                                 +------------------------------+   
                                 |                              |   
                                 |        Browser Cache         |   
                                 |                              |   
                                 +------------------------------+   
                                                                
                                 +-----------------------------+    
                                 |                             |    
                                 |             CDN             |    
                                 |                             |    
                                 +-----------------------------+    
                                                            
                                 +-----------------------------+    
                                 |                             |    
                                 |      Proxy Cache(Nginx)     |    
                                 |                             |    
                                 +-----------------------------+    
                                                                
                                 +-----------------------------+    
                                 |                             |    
                                 |  Distributed Cache(Redis)   |    
                                 |                             |    
                                 +-----------------------------+    
                                                                
                                 +-----------------------------+    
                                 |                             |    
                                 |      In Memory Cache        |    
                                 |                             |    
                                 +-----------------------------+    
                                                        
                                 +-----------------------------+    
                                 |                             |    
                                 |         Database            |    
                                 |                             |    
                                 +-----------------------------+     

As a back end developer, mostly we focus on `Distributed Cache` and `In memory cache`.

### cache Patterns

#### Cache aside
Application will operate DB and cache the same time. as following fig shows:
![cache aside](/img/cache_aside.png)
1st, application first query cache, if hit, then use the data from cache  
2nd, if 1st step not hit, query database directly, return the data  
3rd, after 2nd step, set the cache with data from database to cache

![cache aside read write](/img/cache_aside_read_write.png)


#### Read/Write through
normally application only interact with cache, it is the cache proxy's responsibility to sync data between cache and database.

![read write through](/img/read_write_through.png)

#### Write behind
different from read/write through, data is sync to db from cache through cron job or message queue.

![write back](/img/write_back.png)

For read-heavy task, `cache aside` and `read/write through` is a better choice, for write heavy task, `write behind` is a better choice.

### 3 major problems and solutions in cache world

1, cache avalance  
massive cache keys are expired or cache system down, thus lots of cache missing, then goes to database.  
set different ttl for different keys, or deploy cache system as cluster.

2, cache breakdown  
a cache key didn't hit, thus goes to database. sometimes this key will corresponds to high concurrency. thus lots of query goes to database.  
set a gloable lock when query database for same key to gaurateen only a query goes to database.

3, cache penetration  
the key not existing in cache, thus whenever you query, it will miss.  
set this key to cache with value None.
