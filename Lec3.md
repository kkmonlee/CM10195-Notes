# Inter-Process Communication
<div style="text-align: right">Thursday, 03. March 2017 3:15pm </div>

## Application Level
We briefly look at IPC at the application level, namely high level mechanisms for passing data between processes

- At base, this goes via the kernel (often using a mechanism using pipes, shared memory; assisted by signals or semaphors)

These cam back into prominence with windowing GUIs where it was fond necessary for applications to communicate with each other and with the system.

- Cut-and-paste and drag-and-drop are basic examples, where structured information needs to pass between components (processes)
- The idea is much older than GUIs. This was originally known as a ***software bus***.

Popular implementation include:

- CORBA
- DCOP
- Bonobo
- D-Bus
- COM (Component Object Model) including .NET

These interfaces try to be language- and machine-independant with each language having a set of *bindings* (standard functions) to access them.

- They focus on passing objects between components

So they need a standardised method of representing the data/objects in a *message*.

This is called *message parsing*, another important paradigm for IPC.

These frameworks are very complicated as they need to support a wide variety of communications between a wide variety of computers.

- For example, passing a picture from a program written in C to one writter in Java

Exercise
: Read up on some of these

---
Any of these mechanisms can be used in tandem

- Our program might employ D-Bus to pass data from one application to another (e.g., cut and paste)
- D-Bus might use pipes to communicate between processes
- And pass a filename between them
- and the data is communicated in the file

They are **not mutually exclusive**!

### Which IPC mechanism to choose?
It depends on the application
The best way to choose is to have lots of experience and using them (just like knowing what Data Structure to use)

- The level of your program: is it low or high?
- Amount of data to be communicated: just a bit or a huge datafile?
- What is actually available from the system? (if you don't have a bus, don't use it)
- What your boss tells you to use
- and so on

# Memory
## Memory management
In the earliest of computers, the purpose of memory management was to share out a very limited resource, but it was soon found that inter-process *protection* was vital.

Both needs are still true, particularly the limited aspect: you might have 2GB in your PC, but it's not enough!

> Gates' Law: programs double in size every 18 months

>*(Really Wirth's Law: software is decelerating faster than hardware is accelerating)*

### Physical memory
We first consider how processes (code and data) should be laid out in memory...

This is called *physical memory* layout to distinguish it from *virtual* memory, which comes later.

Memory in a process needs to be allocated or freed at several points

- Allocation at process initialisation. Called *static allocation*
- Allocation while the process is running. Called *dynamic allocation*.
	- Early systems did not support this and you had to know in advance how much memory your process would need at initialisation
- Freeing while the process is running
	- This is dynamic freeing (when a process doesn't need the memory anymore, it is freed up whilst the process is still running
- Freeing at the process end

The kernel also needs memory:

- Allocation and freeing within the kernel. 
The kernel has to be dynamic otherwise it would be very difficult to get started, e.g., creating processes

Early operating systems were not dynamic.

- So they could only run a fixed number of processes
- And the processes were of a fixed size
- Early computer languages did not support dynamic allocation, e.g., FORTRAN, every array must be of a fixed size, declared in the code
- Dynamic allocation for both kernel and the processes was soon introduced in OSs, but computer languages took a while to catch up with the new facility

These days dynamic allocation is common in languages.

- Implicit memory management, e.g., Java.
Where the language controls the creation and deletion of objects
```
bigobject x;	 // memory is allocated
x = foo(); 	// that memory is now inaccessible
```

- Explicit memory management, e.g. C.
Where the programmer controls the creation and deletion of objects (`malloc` and `free`)

There are good and bad points to both approaches.

Physical memory in an early computer with dynamic allocation might have looked like this:

INSERT IMAGE

Simplified Memory Layout with Dynamic Allocation

Remember the kernel itself needs code and data space

- A gap above the kernel area allows for dynamic allocation of memory to itself
- A gap within the process allocation allows for the growth of the stack and dynamic allocation of process data space
- But a gap represents unused and possibly wasted space
- So the OS needs to manage the allocations very carefully

But the earliest sysems had no dynamic behaviour at all.
What they used to do instead was use *partitioning* and *overlays*.

#### Partitioning
The earliest and simplest memory layout is a static system called *partitioning*, where areas are allocated at boot time

INSERT IMAGE

A process is loaded into the smallest free partition it will fit into.

If you don't have dynamic allocation even in the kernel (e.g., for PCBs), then having fixed partitions is ideal.

- Equal size is easy to implement, but usually causes wasted space when a proces does not fill its allocation
- And it can't cope with larger processes
- Variable size is not much harder to implement, but efficiency depends heavily...

Paritioning is a good arrangement if you only run a fixed set of applications that you know in advance, e.g., a stock manager plus a payroll system plus employees record system.

- IBM's OS/360 (mid 1960s) had three partitions

#### Overlays
In early systems, if a process was too big to fit in the memory allocated, the programmer could use *overlays*.

- This is where only *part* of the process code is loaded into memory at once: only partly *resident*
- If a non-resident part of the process is needed, the programmer must know this and include code to loadf the needed part of the process into memory, overwriting a part of the process they do not need at the moment
- If that part of the process is needed again later, the programmer has to reload the code

This works, at the cost of some speed of execution, but only if you are an excellent programmer who can keep track on what parts of code are loaded at any particular time
A similar trick works with data: but newly generated data you have to save it somewhere (e.g., disk), rather than just overwriting it as it's not already there on the disk
This trick of swapping in and out of memory will be seen later.

We need to fit a process into a single contiguous chunk of memory as we can't spread it amongst several areas since

- it will be very complicated for the OS to keep track of which areas of memory are allocated to which process
- more importantly, you can't split code up in this way, having one instruction in one place and the next instruction somewhere else entirely
- similarly for data: we will have to keep track of what data is where

But when we come to virtual memory...

#### Dynamic Partitioning
The next step is to be dynamic: create and allocate a partition as needed

- A lot more complicated to implement, but this allows the process (i.e., the job submission) to say how big a partition it needs and the OS allocates just that

We can allocate sequentially, moving up memory

INSERT IMAGE

The problem is when a process ends and we get memory back: it creates holes

INSERT IMAGE

We have space enough to run a process of size 5, but nowhere to put it.

This is a general problem, called *fragmentation* of memory.

We need to keep a list of free blocks so we can track free space: a linked list called *freelist*
When a block is freed, put it in the freelist. It helps to keep the freelist sorted.

INSERT IMAGE

Slightly more clever is to *coalesce* physically adjacent blocks

When we want some space, we search the freelist.

- We don't want to waste space, so after choosing a big enough block we slice off the chunk we need and return the unused part to the freelist
- But there might be several blocks on the freelist that we could use: which one to choose?
Strategies for choosing blocks include:
	- Best Fit Algorithm. Find the smallest available big enough hole. Slow as we always have to search the entire freelist and results in lots of small frafments that are effectively useless as they are too small to be allocated
	- First Fit Algorithm. Use the *first* available big enough hole. Initially faster than Best Fit and tends to leave larger and more useful fragments. But fragments tend to created near the front of the freelist, so we have to search further and further each time
	- Worst Fit Algorithm. Find the *biggest* available big enough hole. Strangely this works out better than you think. Slicing chunks off bigger blocks tends to leave larger fragments that are more likely to be useful. Marginally faster than Best Fit as we have larger and therefore fewer blocks in the freelist to search through
	- Next Fit Algorithm. Continue looking from where we last allocated and take the next available big enough hole. Fast, and improves on First fit by spreading small fragments across memory
	- And many others


