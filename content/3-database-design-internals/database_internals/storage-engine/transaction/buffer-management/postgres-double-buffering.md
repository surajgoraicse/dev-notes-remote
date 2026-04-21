---
title: Why PostgreSQL uses the Kernel Page Cache While Other Databases Bypass It ?
description:
draft: false
tags:
  - postgres
  - sql
  - database
  - DBI
  - database-internals
modified: 2026-04-16
created: 2026-04-16
---

Most Database engineer will say you that the storage engines must maintain absolute control over memory state. Relying on the operating system's general-purpose cache risks unpredictable eviction of critical index blocks, leading to severe latency spikes during query execution. 
Consequently, high-performance systems like MySQL (InnoDB) and distributed stores like ScyllaDB bypass the operating system entirely to manage memory deterministically. 

Yet, PostgreSQL explicitly violates this principle. It deliberately utilizes a double-buffered architecture, caching data within both its **internal memory space** and the **operating system**'s page cache. This is a calculated engineering trade-off optimizing for ecosystem compatibility, crash recovery simplicity, and kernel-level stability over isolated memory control.


![[Pasted image 20260421120136.png]]

## The Mental Model

To understand why bypassing the kernel is the standard, one must define the standard input/output execution paths in POSIX-compliant operating systems.

- **Standard Buffered I/O:** When an application executes standard `read()` or `write()` system calls, the data is not directly transferred to physical storage. The kernel intercepts the operation and buffers the data within the **kernel page cache** (a portion of system RAM managed by the OS). 
```c
open("data.bin", O_RDONLY);
```

- **Direct I/O :** By utilizing the `O_DIRECT` flag during the `open()` system call, an application instructs the kernel to bypass the page cache entirely.
```c
open("data.bin", O_RDONLY | O_DIRECT);
```

![[Pasted image 20260421095754.png]]


## Postgres Memory Geometry 

PostgreSQL allocates a fixed block of shared memory upon startup known as **shared_buffers**. When a transaction requests a data block (typically 8KB), the system first searches this internal buffer pool.

If a cache miss occurs, Postgres issues a standard POSIX `read()`. Because it does not use **O_DIRECT**, this read hits the **kernel page cache**. If the Linux kernel has the block cached, it is copied directly into **shared_buffers**. This creates the double-buffering phenomenon: the exact same 8KB data block resides in the kernel's memory space and the database's memory space simultaneously.

Systems that avoid this must handle the immense complexity of Direct I/O.




## Answering the Why : 


PostgreSQL uses the kernel page cache because it is designed as a **two-layer caching system**: PostgreSQL keeps a shared database buffer cache in `shared_buffers`, and the operating system keeps another cache beneath it.

This architecture gives Postgres a **second-level cache for free**, keeps the architecture portable, and lets the OS help with read caching and dirty-page writeback while PostgreSQL focuses on transaction correctness, WAL, and buffer management. That is the balance PostgreSQL chooses.

PostgreSQL’s double-buffered design trades theoretical memory efficiency for robust, battle-tested kernel optimizations. By offloading I/O scheduling, read-ahead heuristics, and the bulk of read-caching to the Linux kernel, Postgres maintains a strictly focused internal codebase and inherits decades of OS-level storage enhancements. 

Conversely, systems employing **O_DIRECT** assume the absolute burden of writing custom I/O schedulers and advanced LRU eviction algorithms. They achieve deterministic, ultra-low latency, but at the cost of immense implementation complexity and operational rigidity.

## Why other databases avoid it ?

The double-buffered architecture introduces immediate engineering costs. Memory overhead is the most quantifiable metric; storing identical pages in both user-space and kernel-space reduces the absolute volume of unique data the hardware RAM can retain.
Furthermore, delegating caching to the operating system complicates ACID durability. 

