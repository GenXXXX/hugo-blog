---
title: Operating Systems 知识速记
date: 2017-12-13
showDate: true
tags: ["Operating Systems"]
---

# 0. Concepts

**Process**: Program under execution

**Thread**: Smallest sequence of programmed instructions

**Virtual Memory**: the memory as viewed by the process. Each process believes it has a contiguous chunk of memory starting at location zero (Truman's virtual memory).

**Semaphore**: a [variable](https://en.wikipedia.org/wiki/Variable_(programming)) or [abstract data type](https://en.wikipedia.org/wiki/Abstract_data_type) used to control access to a common resource by multiple [processes](https://en.wikipedia.org/wiki/Process_(computing)) in a [concurrent](https://en.wikipedia.org/wiki/Concurrent_computing) system

**Deadlock**: A deadlock occurs when every member of a set of processes is waiting for an event that can only be caused by a member of the set

**MMU**: Memory Management Unit, translate virtual addresses into physical addresses

**Page**: The program is logically divided into fixed size pieces called pages

**Frame**: The physical memory is similarly divided into fixed sized pieces called frames.

**Page Table**: Table stores the mapping from page to frame

**Demand Paging**: In a system that uses demand paging, the operating system copies a disk [page](https://en.wikipedia.org/wiki/Paging) into physical memory only if an attempt is made to access it and that page is not already in memory

**TLB**: Translation Lookaside Buffer, part of MMU, memory cache stores recent translations from virtual memory to physical memory. The index field is page number, the other fields include frame number, dirty bit, valid bit, etc.

**PRA**: Page Replacement Algorithms. 

**Resident Set**: Data of process that is running which is already loaded to memory

**Page Fault**: Page which is needed is not in the resident set, OS need fetch it from virtual memory or hard drive

**Thrashing**:  OS spends too much time on solving page faults

**Critical Section**: a critical section is group of instructions/statements or region of code that need to be executed atomically 

**Spooling**: 假脱机, a specialized form of [multi-programming](https://en.wikipedia.org/wiki/Computer_multitasking) for the purpose of copying data between different devices. e.g. 打印缓存

# 1. Process v.s. Thread

> Both processes and threads are independent sequences of execution. The typical difference is that threads (of the same process) run in a shared memory space, while processes run in separate memory spaces. (StackOverflow)

The greatest difference is that Process has its own memory space, threads of one process can not be seen by other processes. Process A can not modify the memory space of process B by passing addresses. 

Process can have multiple threads. 

- User level thread: efficient, does not require kernel space, low concurrent efficiency
- Kernel level thread: applications used kernel API to manage these thread, high concurrent efficiency

## Context Switch

When switching the context, OS needs to store the state of current process (pointers in memory space, instructions that finished), load the state of the next process and run the next process.

## System Call

The way that programs ask for service from the OS. e.g. visit the hard drive, create new process.

## Semaphore/Mutex

> In [computer science](https://en.wikipedia.org/wiki/Computer_science), a **semaphore** is a [variable](https://en.wikipedia.org/wiki/Variable_(programming)) or [abstract data type](https://en.wikipedia.org/wiki/Abstract_data_type) used to control access to a common resource by multiple [processes](https://en.wikipedia.org/wiki/Process_(computing)) in a [concurrent](https://en.wikipedia.org/wiki/Concurrent_computing) system such as a [multiprogramming](https://en.wikipedia.org/wiki/Computer_multitasking) operating system.

```basic
loop forever
     P
     critical-section
     V
     non-critical-section
```

**P(s)**

s is a semaphore

```c
s.value = s.value - 1
```

若s.value ≥ 0，则进程继续执行，否则（即s.value < 0），则进程被阻塞，并将该进程插入到信号量s的等待队列s.queue中

说明：实际上，P操作可以理解为分配资源的计数器，或是使进程处于等待状态的控制指令

**V(s)**

```c
s.value = s.value + 1
```

> Semaphore 用于控制允许多少进程/线程同时访问一个critical section. Mutex相当于只允许一个进程/线程访问的semaphore

## Deadlock

4 Necessary conditions:

1. Mutual exclusion: 一资源不侍二主, no sharing
2. Hold and wait: 你有卧室，你想用洗手间，我在用，你得等
3. No preemption: 江山是朕的，朕不给你，你不能抢
4. Circular wait: Each member in the chain is waiting the resource hold by another member of the chain

## Inter-Process Communication, IPC

1. Shared memory + Semaphore

   不同进程通过读写操作系统中特殊的共享内存进行数据交换，进程之间用semaphore实现同步。

2. Message passing

   进程在操作系统内部注册一个port，并且监测有没有数据，其他进程直接写数据到该port。该通信方式更加接近于网络通信方式。事实上，网络通信也是一种IPC，只是进程分布在不同机器上而已。

## Banker's Algorithm

1. If there are no processes remaining, the state is **safe**.
2. Seek a process P whose maximum additional request for each resource type is less than or equal to what remains for that resource type.
   - If no such process can be found, then the state is **not safe**.
   - If such a P can be found, the banker (manager) knows that, if it refuses all requests except those from P, then it will be able to satisfy all of P's requests.
     **Question**: Why?
     **Answer**: Look at how P was chosen.
3. The banker now pretends that P has terminated (since the banker knows that it can guarantee this will happen). Hence the banker pretends that all of P's currently held resources are returned. This makes the banker richer and hence perhaps a process that was not eligible to be chosen as P previously, can now be chosen.
4. Repeat these steps.

## Interrupt

系统调用和中断的关系就在于，当进程发出系统调用申请的时候，会产生一个软件中断。产生这个软件中断以后，系统会去对这个软中断进行处理，这个时候进程就处于核心态了。

## Scheduling

1. FCFS, simple, fair, not efficient
2. SJF, least average waiting time, not realistic
3. SRTF, preemptive SJF, but may starve a process requires long burst
4. Round Robin, preemptive FCFS, quantum as counter for each process, quantum too small, too much switching time; quantum too large, FCFS
5. Priority Scheduling, starve low priority process

# 2. Memory Management

## Page Replacement Algorithms

1. FIFO
2. Optional, pages are replaced which are not used for the longest time in the future, not realistic
3. LRU, least recently used

## Belady’s anomaly

Belady’s anomaly proves that it is possible to have more page faults when increasing the number of page frames while using the First in First Out (FIFO) page replacement algorithm.  For example, if we consider reference string 3, 2, 1, 0, 3, 2, 4, 3, 2, 1, 0, 4 and 3 slots, we get 9 total page faults, but if we increase slots to 4, we get 10 page faults.

>  Multiprogramming requires that we protect one process from another. That is, we need to translate the **virtual addresses** (a virtual address is the address as written in the program) into **physical addresses** (a physical address is the actual memory address in the computer) such that, at any point in time, the physical address of each process are disjoint. The hardware that performs this translation is called the **MMU** or **Memory Management Unit**. 

---

**References:**

[1] http://wdxtub.com/interview/14520847747820.html

[2] http://www.geeksforgeeks.org/operating-systems/
