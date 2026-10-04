---
title: Virtual Memory
source: https://blog.codingconfessions.com/p/virtual-memory
created: 2026-10-03
tags:
---
Normally, we think of virtual memory as a system that provides isolation a the memory level to processes. It also performs some other functions like follows:

- Lazy allocation of memory through demand paging
- Copy-on-write for shared memory between processes, and fast process creation via fork 
- file I/O that avoids the page-cache-to-user-buffer copy using mmap 
- page reclaim, swap, and the page cache 
- performance effects from access patterns, huge pages, TLB shootdowns, and NUMA placement 

Things to know when reading further:

- Alloca is a newly created process.
- And you know what kernel (bridge between software and hardware) is.

## The Need for Virtual Memory 

At any particular time, a very large number of processes are running on any machine and each process have its own virtual memory. Virtual memory feels like real memory, and it consists of addresses that a process can read and write. A process can freely navigate this memory without worrying if it would mess up other processes data. Every virtual address is mapped to a physical address that kernel knows about and gives access in a certain way that no two processes end up sharing the same physical addresses. 

## Size of Virtual Memory 

If the memory is virtual, is it infinite? Not infinite but very large. In x84-64 systems, addresses are stored in 64-bit registers, out of these 64-bit only 48 bits are used due to historical reasons (can be expanded, if future demands). So you can have $2^{48}$ bytes of addressable space, which is 256 TiB (around 256 TB). But a process can't use all of it. Only 128 TiB is available to process and another 128 TiB is used by kernel to provide necessary things to processes. Many modern machines uses around 16GB (or more) RAM but virtual memory is still the same for everyone. 

## The Virtual Memory Address Space layout

How does the 128 TiB memory space given to process look like:

![](../../assets/d2833505-2450-46a4-b342-a7431eb29083_1360x2240.webp)

### What each part does?

- **Text (code) segment**: The compiled instructions of the program. Loaded at startup, mapped read-only and executable. The process cannot write to its own code pages.
- **Data segment**: Global and static variables that have been explicitly initialized.
- **BSS segment**: Global and static variables that are zero-initialized. The binary stores no data for this region; the loader provides zero-initialized memory for it at startup.
- **Heap**: The region for dynamic memory allocation (malloc/new). It grows upwards. You can only allocate memory here programmatically.
- **Memory-mapped region**: A large, flexible area in the middle of the address space used for shared libraries, file mappings, and anonymous large allocations. Libraries like `libc` are loaded here.
- **Stack**: Holds the call frames of all currently executing functions. Starts near the top of the address space and grows downward. Each function call pushes a frame containing local variables, saved registers, and the return address; each return pops it. By default it is 8MB.

### Why such a layout?

It due to two reasons: performance and security. 

- **Performance**: Normally, when you're executing instructions in your code, you typically run them one after another (except for loops and function calls, it's mostly sequential). This pattern of accessing nearby memory location is so common that the hardware is designed around it. Fetching data from physical memory is slow (can take hundreds of cycles), so CPU uses very fast and small cache present right on the chip. When you read a value from memory, hardware doesn't just fetch that value but that entire block. It just fetches nearby values thinking that you would access those only.
- **Security**: Hardware uses something called Write XOR Execute. Memory can be writable or executable, but not both. Heap and stack are read-write but you can't execute here. Text segment is read-only and executable. We achieve this by giving each segment permission bits. 

> ### 🧠 Anonymous Memory
>
> The kernel manages two kinds of memory:
>
> | Type | Description |
> |------|-------------|
> | **Anonymous** | Allocated using `malloc()` or `mmap()`. Used for dynamic program data. |
> | **File-backed** | Memory backed by a file. |

