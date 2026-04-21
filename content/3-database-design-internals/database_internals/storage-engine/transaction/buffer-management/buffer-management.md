---
title: Buffer Management
description:
draft: true
tags:
modified: 2026-04-21
created: 2026-04-21
---
## Note : 

> [!PDF|255, 208, 0] [[Database Internals.pdf#page=144&annotation=4112R|Database Internals, p.144]]
> virtual disk [BAYER72]. A virtual disk read accesses physical storage only if no copy of the page is already available in memory.
> page cache == buffer pool == virtual disk


> [!PDF|255, 208, 0] [[Database Internals.pdf#page=144&annotation=4115R|Database Internals, p.144]]
> The problem of caching pages is not limited in scope to databases. Operating systems have the concept of a page cache, too. Operating systems utilize unused memory segments to transparently cache disk contents to improve performance of I/O syscalls.
> OS also caches the disk content during IO calls.


> [!PDF|187, 97, 229] [[Database Internals.pdf#page=144&annotation=4118R|Database Internals, p.144]]
> any changes are made to the cached page, it is said to be dirty, until these changes are flushed back on disk.





