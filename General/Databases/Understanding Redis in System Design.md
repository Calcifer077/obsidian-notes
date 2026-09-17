---
title: Understanding Redis in System Design
source: https://regoo707.medium.com/understanding-redis-in-system-design-7a3aa8abc26a
created: 2026-09-15
tags:
  - database
---
## What is Redis?

Redis is an open-source, in-memory data structure store, used as a database, cache, and message broker. Redis can be used as a traditional monolithic and can be used as a distributed system as a cluster of nodes with sharding. 

## What is memory caching?

A **cache** is like short-term memory. Accessing data from memory is faster than from a hard disk. Caching means saving frequently accessed data in-memory so the value that is added by caching is retrieving data fastly and reduce calling the original data source. 

