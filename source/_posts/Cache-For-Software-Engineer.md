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
### cache Patterns

#### Cache aside

#### Read/Write through

#### Write behind

### 3 major problems and solutions in cache world

