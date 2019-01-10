# Overview | 概述

Application memory management involves supplying the memory needed for a program’s objects and data structures from the limited resources available, and recycling that memory for reuse when it is no longer required. Because application programs cannot in general predict in advance how much memory they are going to require, they need additional code to handle their changing memory requirements.

## Tasks | 任务

Allocation

When the program requests a block of memory, the memory manager must allocate that block out of the larger blocks it has received from the operating system. The part of the memory manager that does this is known as the allocator. There are many ways to perform allocation, a few of which are discussed in Allocation techniques.
Recycling

When memory blocks have been allocated, but the data they contain is no longer required by the program, then the blocks can be recycled for reuse. There are two approaches to recycling memory: either the programmer must decide when memory can be reused (known as manual memory management); or the memory manager must be able to work it out (known as automatic memory management).

## Metric | 指标

CPU overhead

The additional time taken by the memory manager while the program is running.
Pause times

The time it takes for the memory manager to complete an operation and return control to the program.

This affects the program’s ability to respond promptly to interactive events, and also to any asynchronous event such as a network connection.

Memory overhead

How much space is wasted for administration, rounding (known as internal fragmentation), and poor layout (known as external fragmentation).

# Challenge | 挑战

The basic problem in managing memory is knowing when to keep the data it contains, and when to throw it away so that the memory can be reused.

Premature frees and dangling pointers

Many programs give up memory, but attempt to access it later and crash or behave randomly. This condition is known as a premature free, and the surviving reference to the memory is known as a dangling pointer. This is usually confined to manual memory management.
Memory leak

Some programs continually allocate memory without ever giving it up and eventually run out of memory. This condition is known as a memory leak.
External fragmentation

A poor allocator can do its job of giving out and receiving blocks of memory so badly that it can no longer give out big enough blocks despite having enough spare memory. This is because the free memory can become split into many small blocks, separated by blocks still in use. This condition is known as external fragmentation.
Poor locality of reference

Another problem with the layout of allocated blocks comes from the way that modern hardware and operating system memory managers handle memory: successive memory accesses are faster if they are to nearby memory locations. If the memory manager places far apart the blocks a program will use together, then this will cause performance problems. This condition is known as poor locality of reference.
Inflexible design

Memory managers can also cause severe performance problems if they have been designed with one use in mind, but are used in a different way. These problems occur because any memory management solution tends to make assumptions about the way in which the program is going to use memory, such as typical block sizes, reference patterns, or lifetimes of objects. If these assumptions are wrong, then the memory manager may spend a lot more time doing bookkeeping work to keep up with what’s happening.
Interface complexity

If objects are passed between modules, then the interface design must consider the management of their memory.

## Maunal & Atomic

Manual memory management is where the programmer has direct control over when memory may be recycled. Usually this is either by explicit calls to heap management functions (for example, malloc and free(2) in C), or by language constructs that affect the control stack (such as local variables).

some manual memory managers perform better when there is a shortage of memory. 不过开发者必须编写大量的额外的内存管理相关的代码，并且极有可能过量分配内存或者造成其他的错误。

Automatic memory management is a service, either as a part of the language or as an extension, that automatically recycles memory that a program would not otherwise use again. Automatic memory managers (often known as garbage collectors, or simply collectors) usually do their job by recycling blocks that are unreachable from the program variables (that is, blocks that cannot be reached by following pointers). memory may be retained because it is reachable, but won’t be used again;
automatic memory managers (currently) have limited availability.
