---
tags:
  - pds
  - os
  - s2
  - 2023-2024
  - os161
---
# Memory Management 
#2024-03-4

## Memory Management Unit - MMU


## Page Table
### Inverted Page Table
#### Architecture
Single page table for all processes. every entry have a `pid,p,d` where
- `pid`: process identifier
- `p`: page number 
	- (conversion from logical address to physical one)
- `d`: offset from the page address
	- remain intact as the dimension of the page is equal from the page table to the physical memory

---
# OS161

## Outline
- [[2023-2024/Semester II/Programmazione di Sistema/OS Internal/LAB OS161/LAB#Lab00 - Setup VM with OS161|setup]]
- Understanding
- Building
- Running
- Kernel threads
	- thread library
	- user processes
	- context switch
- ***something***call
## Introduction
#2024-03-11

OS 161 is a simplified operating system written in c, targeting students.
It was released in 2015.
Main features of this os are the high accessibility done to let students deeply understand how an os works.
### framework
os161 includes:
- sources of the kernel to be used for:
	- code browsing
	- designing
	- running and debugging
- toolchain for
	- cross compiling
	- running kernel on top of a simulator (**sys161**)
	- others
### About System/161
Sys161 is a machine simulator that provides a simplified but still realistic environment. Target a 32-bit MIPS system supporting up to 32 processors (need a virtual machine!). It also supports transparent debugging via remote gbd into simulator
![[Pasted image 20240311131638.png]]

### Os161 Support
It is simplified, so not all the features of a regular operating system are implemented. This is done so the students can add their implementation to really understand how an os work, and thus realizing a personalized os. 
What can be implemented?
- Locks
- system calls
- virtual memory
	- `dumbvm` already present is good enough for bootstrapping and doing early assignments. The problem is that it never reuse memory, so cannot support large files.
- file system
- and more!

### Working os161
main directory:
![[Pasted image 20240311132304.png]]

### Building
go to  `cd ~/os161/root/` run `sys161 kernel`

There are 3 ways to debug:
- terminal
- gui
- 2 terminals
### Making os161 new release
After implemented new features build new kernel.
Three main directories:
![[Pasted image 20240311133002.png]]

`conf` used to configure new kernel
`compile` to compile it (DUH!)
workflows commands for building:
- `./config Hello`
- `bmake depend`
- `bmake install`

![[Pasted image 20240311133152.png]]

### Making new Release
Start by copying the dumbvm kernel which is known to work, so starting from a copy of a source that works. Two advantages:
- starting from something that works
	- Every error is an error from the students
- with a copy it's easy to roll back if too many errors arise

inside `kern`
`cp DUMBVM HELLO`
`./config HELLO`

`bmake install`


## What is a thread (Process)?
Thread represents the control state of an executing program
has an associated context: 
- CPU state:
	- program counter
	- stack pointer
- other

## User threads and kernel threads
- User:
	- management done by user-level threads library such as:
		- posix pthread
		- windows threads
		- java threads
- Kernel threads - supported by the kernel:
	- Windows
	- Linux
	- Mac OS X
	- iOS
	- Android

The threads must be mapped, but how?
![[Pasted image 20240311134730.png]]

- One-to-one: mapping each user thread to a kernel thread
	- ![[Pasted image 20240311134837.png]]
	  cons: limitation on number of threads that can be created
- Many to One: mapping many user level threads to one kernel thread
	- ![[Pasted image 20240311134943.png]]
	  Cons: user threads must wait their turn
- Many to many: mapping many user level threads to $<=$ number of kernel threads:
	- ![[Pasted image 20240311135037.png]]
	  best options

### Process Concept
An os exec many programs that run as a process
Process: a program in execution, process execution must progess in a sequential fashion
- program code: text section
- current activity (PC, PR)
- stack
- data section
- heap

![[Pasted image 20240311135404.png]]

Memory Layout in C Program
![[Pasted image 20240311135553.png]]
program is passive when ===...===

when multithreading code, data and files are in common 

### Threads in os161

#### Single thread
memory: stack, data, text
uses also registers

#### Multiple threads

##### User level
There is a thread library for this: `kern/include/thread.h`
The data structure are memorized in Thread Control Block (***TCB***)

different properties:
- name
- state
- etc.

there is a tcb pointer that point to the data structure

different data structure for different purpose:
- `switchframe`: allow os to save registers of the cpu associated to the thread. When switching the os saves all registers in this data structure as part of kernel memory.
- `cpu`: in which cpu the thread run.
- `process`: each thread know its process, while the process does not know its thread.
	- It is possible to implement the knowledge of the thread from the process.
- 

##### Kernel level
In the os161 kernel threads implementation, thread context are stored in thread structures.

### Thread Library and Two Threads (generic)
processor with 2 active threads:
1. having two stacks the the threads
2. thread library: on of the two is active
	- inactive one should save its context and register in its stack
3. switching thread in os:
	- recovering data from stack
	- load into cpu
	- all of this while saving the active thread in its stack

#### recap
os161 we have a memory space to be managed to allocate code, data, and stack (**num of stacks = num of threads**)
In case of multiple threads data and code are shared between the threads.

### OS161 Thread Library

#### Thread Structure
#2024-03-13
There are two elements to allow saving the context in thread: kernel level stack and switch frame

In the example of two thread: `stack 1` active and `stack 2` inactive. 
`stack 2`'s `switchframe` have stored the information about the inactive thread (PC, SP, etc..).

#### Thread Interface
In os161 there are multiple function to interact with thread:
-  [[#`Thread_fork`|Thread_fork]]: create new thread
	- First step, defining an entry point which is a pointer to a zone of a memory (third parameter of the `thread_fork` function, a pointer to the address of a memory that contains the codes of the processor). In os161, the entry point has a fixed structure, expecting two parameters: 
		1. a pointer to the data memory (the function to execute). 
		2. an integer number (a pointer to the parameters of the function)
- `thread_exit`: exit/disable thread
- `thread_yields`: context switch between two thread
	- Force the context switch, calling this function pause the thread and ask the OS to schedule the next thread
- `thread_switch`: change the state of the current thread, taking a new one from the list
	- Calling this function will change the context

##### `Thread_fork`
In reality `Thread_fork` is actually a wrapper for 3 functions:
1. `thread_create`:
	- Create the thread, allocate the TCB and the necessary memory for managing the memory 
2. `switchframe_init`:
	- Create the switch frame allowing to memorize the values of all the registers to initialize the frames
3. `thread_make_runnable`:
	- Change the status of the thread to ready and insert the thread it to the ready list to be scheduled

### Application (User Process) and Kernel
Until now it was all around the kernel point of view of how to manage different threads. Nonetheless, the logic for an user application remain very similar. The main difference where in the kernel every logic has to manage his thread, now there is a "manager" for user application thread. The user does not implement the kernel code, it is already implemented at the kernel level how to manage those process. This unit is called Process Control Block: #PCB.
Another difference is the knowledge about the thread from the user perspective. All the threads used for the process knows which process to point but it is not also true for the opposite, the user only know how many thread are associated to him but a 0 knowledge about which specific thread was assigned to him.

#### Running a user program
When executing a program (`p.elf`)
1. OS calls `proc_create_runprogram` to create a user process (create PCB and allocate memory)
2. Each process should have at least one thread, therefore, calling `thread_fork`
3. `run_program` execution: generating the address space, loading the elf file into memory, starting the execution of the thread.

## A view of Operating System Services
What is a system call? No user can talk directly to the operating system (big security risk!), to make possible communication between the different layers, a system call is invoked with a specific id. 
This invocation make an interrupt and from this point there is a switch from user mode to kernel mode, then the id is used to identify the request and execute the request, switch to user mode and let the user know the output of the request.

### OS161 System Call managements
As it's very simple there is no difference between system calls, exceptions and interrupt.

#### OS161 trap frame
Every time a system call is called, the execution is passed from user mode to kernel mode. Therefore, the status of CPU should be saved through trap frame. Trap frame will be saved at the stack of kernel thread. So, while a thread is being executed, it is possible that the execution is interrupted (by system call, exception, interrupt). Therefore, processors state should be saved for the execution of the system call and reloading it after.

#### OS161 MIPS System Call Handler
Handler for system call in MIPS through switch case: When a system call is called, the handler is executed which save the context of the thread in a trap frame, executing the code related to the system call (#)
```c
Void syscall(struct trapframe *tf) { 
.. 
	callno = tf->tf_v0; retval = 0; 
	switch (callno) { 
		case SYS_reboot: 
		err = sys_reboot(tf->tf_a0); /* in kern/main/main.c */ 
		break;
		
		/* Add stuff here */ 
		default: 
		kprintf("Unknown syscall %d\n", callno); 
		err = ENOSYS; 
		break; 
}
```

#### Two Processes in OS161
When there are multiple program in execution:
1. two user memory address for the two application
2. Data, code and stack section for each application
	- data and code section from kernel remain in common
3. Two kernel for the two threads for storing different data structure such as switch frame or trap frame.

#### OS161 Memory Management
In OS161 there are 4GB of total memory in order bottom-up: 
- 2GB for User space
- 2GB kernel space:
	- 0.5GB Kseg0: are not passed through #TLB but cache (direct access to physical address)
	- 0.5GB Kseg1: not passed through #TLB not cache, for mapping I/O devices.
	- 1GB Kseg2: not used in OS161.

---
# Virtual Memory
## Objective
#2024-03-18
Memory management techniques aims to load as many processes as possible at the same time. 
One problem to face is when a single process try to allocate more memory then the one available. This means that the logical memory must depend on the physical one.
If a new process arrives and there is no more free memory space left, one or more process may be moved on external memory (**very slow**). For this reason there must be a way to separate user logical memory and physical memory: ***Virtual Memory***

Only part of the program needs to be in memory for execution, thus if new process arrive and there is no free memory left, there is no need to completely stop a process from running. 

## Virtual Address Space
Usually logical address space for the stack start from the **MAX_VALUE** and then go *down*. Virtual memory allow files in memory to be shared between processes.
![[Pasted image 20240318132039.png]]
This, in the right condition, greatly reduce memory usage.

## Implementation
There are two ways to implement Virtual Memory:
1. [[#Demand paging]] (preferred, the one we'll study)
2. ~~**Demand segmentation**~~ (old and unused)

## Demand paging
First bring all to memory
Load pages only as they are demanded (**Demand Paging**)
- Pages that are never accessed are never loaded into memory

### Basic Concepts
During execution some pages will be in memory, other in secondary storage. But, *how* a process know where a page is? **valid-invalid bit**
The process trying to access the memory will go into his page table searching for the specific page and check the relative valid-invalid bit.
- if the bit is set to ***valid*** the bit is both legal or in memory
- if the bit is set to ***invalid*** the page is not in the logical address space or the page is in the secondary storage
![[Pasted image 20240318132907.png]]
The first page at index 0 has the bit set to valid and thus the frame number will actually point to the physical address.
The second page at index 0 has the **valid-invalid** bit set to *invalid*, but since we know that the addressing is correct, it must be in the secondary memory represented in the image with a cylinder (of course it will have all the pages as it's where the process is loaded from). If an access to a page that was not brought to memory happen it will result in a **page fault***.
If i search the 8th index (although not present in the page table) it would show an *invalid* bit, but this time is because the address is **wrong**! -> **page fault**

### Page Fault
When a page fault exception is thrown the Operating System will follow the following steps:
1. **Reference**: check the page table to find a reference of the page that the process want to access.
2. **trap**: if the reference was invalid (outside logical address of the process) terminate the process. Otherwise (**valid**) the page is brought into memory to let the process access it.
3. **Find a free frame** to load page into.
4. **Bring in missing page** from the secondary storage into the free frame found into the main memory.
5. **Reset page table** to let the new page have a **valid** bit and add a reference for the frame in memory.
6. **Restart instruction** to let the process continue. This time the page fault will not happen as it will now have a valid bit and will access the right address in memory. 
7. ***Hurray!***

### Extreme Case
Start executing a process with ***no*** pages in memory
- os sets instruction pointer to first instruction of process, non-memory-resident => page fault
- **pure demand paging**
A given instruction could access multiple pages -> multiple page faults
- fetch and decode of instruction that adds 2 numbers from memory and store result back to memory
- this would result in unacceptable system performance
- However not likely as programs tend to have ***locality of reference*** (80% of instructions access 20% of memory) which result in reasonable performance from demand paging

### Free-Frame List
When a page fault occurs the os must bring the desired page from secondary memory into the main one. 
To do this, most os maintain a **free-frame list**: a pool of free frame to satisfy this request.
Typically os allocate free memory with using a technique known as **zero-fill-on-demand**: zeros out the frame before allocating them (security reason)

### Performance
To compute the **effective access time** ($EAT$) for a demand-paged memory, let's assume **memory access time** ($T_{ma}$) is 10 \[ns\]. This value is increased in case of page fault with a value known as **page fault time** ($T_{page fault}$).
Let's consider $p$ as the probability of page fault ($0<= p <= 1$) 
$EAT = (1-p) * T_{ma} + p * T_{page fault}$

### Worst Case
1. Trap to the operating system
2. save the user registers and process state
3. 2. Virtual Memory (ch10), page 42
4. while waiting allocate CPU to other user
5. receive an 


### Example
$T_{ma}=200[ns]$


## Copy-on-Write
Process creating using the `fork()` system call may bypass the need for demand paging.
A technique called **copy-on-write (COW)** may allow the parent and child processes to initially share the same pages
- When one of the two modifies a shared page, then, and only then, the page is copied.
	- While the page is used as read only, there are no problem if the page is shared, when write happens, data hazards may arise and there is a need to distinguish the page for the two processes 

## Page Replacement
If we increase the degree of multiprogramming we may find ourself in a ***over-allocating memory*** situation. When process call for a page fault there are no free memory left so a swap happens, but it may continue for different processes and the os may uses a lot of time for continues swapping and thus a continuous page replacement that will have no benefits, instead will negatively impact the overall performance.
- Too much overhead of copying entire process between memory and swap space


## Virtual Memory - Summary
#2024-04-08

Multiple processes in parallel, avoid loading all the data in memory, using only the one needed. This will greatly improve memory footprint and multi processing execution at cost of more complex system to avoid **page fault** to crash the program and little more delay to load the memory when needed (actually not much relevant as the 20/80 rule: 20% of memory will suffice for 80% of the program)

### Handle page fault
take needed page from memory and load it into memory to let the program continue it's execution:
- if there is a free frame in memory: load it into it! 👍
- no free frame in memory:
	- Find a *victim frame*: unused frame to kill. transfer the *victim frame*(the page already loaded into the main memory) to the secondary storage to be saved in case of later access, then load the original page that caused the page fault into the primary location:
		- ==PROBLEM:== VERY SLOW! need two memory transfer for a single memory fault! **double time**
	- **modify (dirty)** *bit* to reduce overhead of page transfer
		- dirty bit is set if the page has been modified, thus if a frame has no dirty bit it can be replaced without transferring it into the external storage

## Page and Frame replacement Algorithms
- **Frame-Allocation Algorithm**
	- Multiple process in memory -> decide how many frames to allocate to each process
- **Page-replacement Algorithm**
	- When page replacement is required we must select the frame that are to be replaced
	- **lowest page-fault rate**


## Page-replacement Algorithm
Select a particular algorithm, but which one? 
- FIFO Page Replacement 
- Optimal Page Replacement 
- LRU Page Replacement 
- LRU-approximation Page Replacement 
- Counting-based Page Replacement 
- Page-Buffering Algorithm 
- Applications and Page Replacement

Evaluate testing multiple algorithm on the same condition. Having a string of memory references (reference string):
- String is just page numbers not full address
- repeated access to same page does not cause page fault
- results depend on number of frames

>In the following example the **reference string** will be: 
>> 7,0,1,2,0,3,0,4,2,3,0,3,0,3,2,1,2,0,1,7,0,1

The image show how increasing the numbers of frame asymptotically reduce the number of page faults, as more space available to to allocate more memory. There will always be a limit as the physical memory will be less than the logical memory (the all point of paging!) but the overhead and the lost time is very low
![[Pasted image 20240408133102.png]]
==THIS IS ONLY HYPOTETICAL==

Of course in the real use case pages are not allocated equally and some pages may need more usage then others.

Empirical probability
$f(A,m) = \sum_{\forall w} p(w)\frac{F(A,m,w)}{len(w)}$ 
A page replacment algorithm
w given reference string
p(w) probability of reference string w
len(w) lenght of reference string w
m number of available page frames
F(a,m,w) number of page faults generate with the given reference string (w) using algorithm A on a system with m page frames

### Optimal Algorithm
Replace page that will not be used for longest period time. Guarantee lowest possible page-fault but difficult as it require to know the future memory access.
Applying the opt with the reference string produce **9** fault

### First-In-First-Out ( #FIFO ) Algorithm
The fifo algorithm add from the bottom and push it to the top (like the queue)

consider the following reference: `1,2,3,4,1,2,5,1,2,3,4,5`
first colum 1,2,3 are not present in memory and thus 3 page-fault, column 2 the index 4 are not present in memory so page-fault and added in memory ad the start, and so on. In the 6th column the index 1 is accessed and thus, as it's already in memory, no page-fault occurs

| 1,2,3 =>❌ | 4 => ❌ | 1 => ❌ | 2 => ❌ | 5 => ❌ | 1 = > ✅ | 2 => ✅ | 3 => ❌ | 4 => ❌ | 5 => ✅ |
| --------- | ------ | ------ | ------ | ------ | ------- | ------ | ------ | ------ | ------ |
| 1         | 2      | 3      | 4      | 1      | 1       | 1      | 2      | 5      | 5      |
| 2         | 3      | 4      | 1      | 2      | 2       | 2      | 5      | 3      | 3      |
| 3         | 4      | 1      | 2      | 5      | 5       | 5      | 3      | 4      | 4      |
![[Pasted image 20240408134435.png]]


Total page fault with 
- 3 frames: 9
- 4 frames: 10

This is the Belady's Anomaly for the fifo algorithm (sometimes **more** frames causes **more** page fault)
![[Pasted image 20240408134118.png]]


Using the reference string produce **15** faults, easy to implement but not ideal

### Least Recently Used ( #LRU ) Algorithm
Consider the last used page and use this information to replace pages. When page fault find the victim frame using the oldest one. Use the past instead of the future

The #LRU produce **12** faults using the reference string. Better then fifo but still worse than the opt

Two possible implementation:
- counters:
	- associate with each page-table entry a time-to-use field and add to the cpu a logical counter
- Stack
	- Keep a stack of page numbers. When page is referenced it's removed from the stack and put on the top

### Second-Chance Algorithm
is a #FIFO replacement algorithm with the additional control of the reference bit:
- bit 0: replace the page
- bit 1:
	- give the page a second chance and search the next #FIFO page
	- in this moment the reference bit is cleared (set to 0), arrival time reset to the current time
- Check next page with same rule

Circular search
![[Pasted image 20240408144517.png]]
### Enhanced Second-Chance Algorithm
Improve using both **reference bit** and **modify bit** possible following cases:
- `(0, 0)`: neither recently used nor modified – best page to replace
- `(0, 1)`: not recently used but modified – not quite as good, must write out before replacement
- `(1, 0)`: recently used but clean – probably will be used again soon
- `(1, 1)`: recently used and modified – probably will be used again soon and need to write out before replacement

When page replacement is called, use the clock scheme but use the four classes to replace the page in the lowest non-empty class
- might need circular queue several times

### Counting-based Algorithms
Keep a counter of the number of references that have been made to each page

- Least Frequently Used (LFU) Algorithm: replaces page with **smallest** count
	- To face the problems of when a page is heavily used during the initial phase of a process and then never used again
- Most Frequently Used (MFU) Algorithm: replaces page with **highest** count
	- with the argument that the page with the smallest counter was just brought it and thus of course will have a small number




# OS161 Memory
#2024-04-15

## DUMBVM and Kmalloc
Memory management of #os161 consists of two elements:
- #Kmalloc: allows memory allocation of kernel side
- #Dumbvm: memory management user side

Memory management uses contiguous allocation, however it also uses pagination (minimum amount of available memory is page(s))
> page dimension is 4KB (4096 byte)

## MIPS virtual address space
os161 is based on #MIPS, a 32 bit processor and addresses at 32 bit. Therefore logical address space of 4GB.
Mips maps the kernel of OS in the logical memory space of the process so that the process sees the user and the kernel in its memory space.
When boot of os161 is done, the kernel is mapped in physical address that later is mapped, through #TLB, in this logical memory of the process.

- kuseg: user memory (2GB)
- kseg0: kernel (0.5GB)
- kseg1: I/O devices (0.5GB)
- kseg2: not used (1GB)
Total: 4GB
## Dumbvm.c (alloc)
Memory management architecture of os161: `ram_stealmem` for memory management at low level, `getppages` for memory management in `dumbvm.c`. There are also function for user side (`as_prepare_load`) and kernel side (`alloc_kpages`)

Layout of physical memory:
Starting from the first free address, arriving to the dimension of the memory, the first free address depends on the size of the kernel, while the last one depends on the ram size

## `ram_bootstrap`
During the boot the `ram_bootstrap` is callled and it will initialize different parameters such as `ramsize` using `mainbus_ramsize`, saving `lastpaddr` as the ram size and calculating the first free address with respect to `KSEG0`

## `ram_stealmem`
in `ram.c` the most important function is `ram_stealmem` that allow to ask for memory. `Npages` is the number of pages to ask the OS. Calculate the size multiplying the numbers of pages with the page size.
```c
size_t size = npages * PAGE_SIZE;
```

And check whether the amount of memory requested can fit in the free physical memory, if there is no space (the address will get out of the physical address), return 0. This will fail the memory allocation
```c
if(firstpaddr + size > lastpaddr) {
   return 0; 
}
```

Otherwise, save the corresponding variable with the first available address in the physical space and the memory allocation is successful
```c
paddr = firstpaddr; 
firstpaddr += size; 
return paddr;
```

## `getppages`
Is used for ask numbers of physical pages, asking for page number and returning pointer to the zone of memory.
`spinlock` allows the mutual exclusion in case of multiple process, to avoid corrupting of `ram_stealmem()`

![[Pasted image 20240415132846.png]]


## #TODO dumbvm memory allocation

The function for allocation of memory is implemented BUT the functions for releasing the memory is NOT!

![[Pasted image 20240415133008.png]]

`free_kpages` and `as_destroy` (equivalent of `alloc_kpages` and `as_prepare_load`) are defined but empty! Need to implement those function.

### De-alloc (free) - Solution A1
`freeppages` fust an interface to `ram_freemem`
Defining the equivalent system of `ram_stealmem` and `getppages`, define a `freeppages` that will be called by `free_kpages` and `as_destroy`

`freeppages` just an interface to `ram_freemem`
Data structure and memory management in `ram.c`

![[Pasted image 20240415133407.png]]



# Paging
## Allocation of Frames
How do we allocate the fixed amount of free memory among various processes?

>If we have 93 free frames and two processes, how many frames does each process get?

- Operating system may take 35 frames, 93 for user space
- Pure demand paging: 93 frame on free-frame list
- First 93 pages fault would all get free frame from the free-frame list
- For a new page demand, a page-replacement algorithm would be used to select one the 93 in-memory pages to be replaced with $94^{th}$
- When the process terminated, the 93 frames would be placed on free-frame list.

> Many variation of this strategy!

If we have 93 free frames and two processes, how many frames does each process get?

Need a minimum amount of frames for each process:
- When a page fault occurs before an execution of the instruction is completed, the instruction must be restarted.
- need enough frames to hold all the different pages that any single instruction can reference

**Minimum**: number of frames per process is defined by the architecture
**Maximum**: total frames in the system

At this point there are two major allocation scheme:
- Equal Allocation
- Proportional Allocation

## Equal Allocation
split **m** frames among **n** processes. Thus each process will receive: $m/n$ frames (ignoring frames needed from the os itself for simplicity )

Not very good as different processes may have different requirement and thus inefficient allocation

## Proportional Allocation
Allocation of available memory to each process according to its size.

Considering a system with 1KB frame size.
If a small student process fo 10KB and an interactive database of 127KB are the only two processes running in a system with 62 free frames, it does not make any sense to give each process 31 frames! (the student process don't need more than 10 frames)

$s_i=$ size of virtual memory for process $p_i$
$S= \sum s_i$ 
$m =$ total number of available frames
Allocate $a_i$ frames to the process $p_i$, such as $a_i$ is approximately $>> a_i = S_i / S * m$
Where $a_i$ should be an integer that is greater than the minimum number of frames required by the instruction set, with a sum not exceeding m.

If the multiprogramming level increases, each process will lose some frames to provide the memory needed for the new process and vice versa
Either in equal or proportional allocation, the priority is ignored during reallocation of memory. However it's possible to give an high priority process more memory to speed its execution.

## Global vs. Local Allocation
With multiple processes competing for frames, we can classify the page-replacement algorithm into two categories:

- **Global replacement**: process selects a replacement frame from the set of all frames even if the frame is currently allocated to some other process
	- Set of pages in memory for a process depends not only on the paging behavior of that process, but also on the paging behavior of other processes.
	- The process execution time can vary greatly.
- **Local replacement**: each process selects from only its own set of allocated frames.
	- The set of pages in memory for a process is affected by the paging behavior of only that process. 
	- More consistent per-process performance.

Generally, the global replacement results in greater system throughput, therefore, it is used more commonly


## Reclaiming Pages
A strategy to implement a global page-replacement policy.
All memory requests are satisfied from the free-frame list rather than waiting the for list to drop before selecting the replacement.
Page replacement is triggered when the list falls below a certain threshold.
This attempts to ensure sufficient free memory always available for new requests.

## Non-uniform Memory Access
Until now we said all memory accessed equally with equal access time.
On Non-Uniform Memory access ( #NUMA), i.e. systems with multiple CPU, that is not the case
- A given CPU can access some sections of main memory faster than others.

A cpu can access its local memory faster that the local memory of another CPU, #NUMA systems are slower than systems in which all access to main memory are treated equally
![[Pasted image 20240415140007.png]]


## Trashing
If a process does not have enough pages (the os didn't gave enough pages when the process started), the **page-fault rate is very high**
- page fault to get page
- replace an active page
- quickly need the replaced page => page fault
- The cycle repeats

A process is ***trashing*** if it is spending more time paging than executing
This leads to:
- Low CPU utilization
- OS thinking that it can increase the multiprogramming
- Another process added to system
- LESS PAGE AVAILABLE FOR PROCESSES
- more **trashing**

![[Pasted image 20240415140345.png]]

How to fix this issue? 

- **Local Replacement Algorithm**:
	- Each process select from only its own set of allocated frames
	- If one process start trashing it cannot steal frames from other processes containing the trashing only in the current process

This will keep the process in the queue for a long time, **increasing the access time** even for a process that is *not* trashing.

**Locality model**:
- The locality is a set of pages that are actively used together
	- A running program is generally composed of several different localities.


## Working-Set Model
**Working-Set Model** is based on an assumption of locality and use a parameter $\Delta$ to define the **working-set window**. The set of pages in the most recent $\Delta$ is **working set**

The accuracy of the working set depends on the selection of $\Delta$
$\Delta$ too small will not encompass entire locality
$\Delta$ too large will encompass several locality
$\Delta=\infty$ will encompass entire program

$WSS_i$ (working set of Process $P_i$)= total number of pages referenced in the most recent $\Delta$ 

Once $\Delta$ is chosen, the OS allocates to the working sets, enough frames to provide it with its working-set size
- enough extra frames: another process can initiated
- sum of working set sizes exceed total number of available frames: #os will select a process to suspend and swap out the pages, reallocating the frames to other processes.
	- Prevent **trashing** while keeping a degree of multiprogramming as high as possible

Assuming $\Delta=10,000$ references
- Timer interrupts after every $5,000$ references
- When timer interrupt is received, copy and clear the reference-bit values for each page, thus if a page fault occurs, in-memory analysis of usage within the last $10,000$ to $15,000$ references

## Page-Fault Frequency
Establish "acceptable" **page-fault frequency** #PFF rate and use local replacement politcy
- #PFF too high: process needs more frames
- #PFF too low: process may have too many frames

Establish upper and lower bounds on the desired page-fault rate.


# Allocating Kernel Memory
#2024-04-26
kernel memory is allocated from a free-memory pool for 2 reasons:
- Kernel allocate data structure of varying size, sometimes even less then a page file size. And so the kernel try to minimize the wasted space and the fragmentation.
- some kernel memory needs to be contiguous. Certain hardware interact directly with the memory without the virtual one.

Primarily two solution:
- Buddy system
- Slab allocation

## Buddy System
Allocates memory from fixed-size segment consisting of physically contiguous pages. Allocate memory with size equals to power of 2.
An allocation is rounded to the next higher power of 2 value.
When smaller allocation needed than is available, current chunk split into two buddies of next-lower power of 2.

> For example, assuming the size of a memory segment is 256KB, kernel requests 21KB

The segment is initially divided into two buddies >> $A_L$ and $A_R$, each 128 KB in size. One of these buddies is further divided into two 64KB buddies >> $B_L$ and $B_R$ from which either $B_L$ or $B_R$ is divided again into two 32 KB buddies >> $C_L$ and $C_R$ .
One of these buddies is used to satisfy the 21KB request.

![[Pasted image 20240426085907.png]]

An advantage of the buddy system is how quickly adjacent buddies can be combined to form larger segments using a technique known as ***Coalescing***
In the previous example, when the kernel release CL unit, the system can coalesce CL and CR into a 64 KB segment
## Slab allocation
(used in unix like system)
Group different memory location in fixed size and then allocate those location.
The **Slab** is made up of one or more physically contiguous pages, **Caches** consists of one or more slabs (not the previous known term for cache!).

But why?
Once a kernel asked for an allocation, the next one will use a different slab, thus minimizing the amount of work done to find free memory

# Address Translation Scheme
Address generated by CPU is divided into:
- Page number (*p*): index for a page table which contains base address
- Page offset (*d*): combined with base address to find the physical address

# Mass Storage
#2024-05-13
The main mass-storage in modern computers is secondary storage, usually provided by **hard disk drives** (HDDs) and **nonvolatile memory devises** (NVM)

## Hard Disk Drives
HDDs are disk platters with two surfaces covered with a magnetic material. The information is stored and read magnetically on the platters by an arm.
The disk rotate while the arm position himself relative to the center to read the different **cylinder**.
![[Pasted image 20240513131135.png]]

Multiple disks are read simultaneously, the vertical volume is called a **cylinder**, the area of a disk in cylinder is called **track**, from which there are multiple **sector** that is the actual position where the arm head read the information.

Useful characteristics are:
- transfer rate: rate at which data flow 
- positioning time / random-access time:
	- seek time: time to move the arm in the required **cylinder**
	- rotational latency: time to rotate the disk to let the arm sense the **sector** magnetic field

## HDD Scheduling
The OS is responsible for using hardware efficiently, for HDDs this means **minimizing** access time and **maximizing** data transfer bandwidth.
Access time:
- minimize seek time
	- seek time ~= seek distance

### Hard Disk Performance
$Access Latency = Average Access Time = Average Seek Time + Average Latency$
Typical timing:
- Fastest disk: $3 + 2 = 5ms$
- Slow disk: $9 + 5.56 = 14.56ms$

Average I/O time = Average Access Time + (amount to transfer / transfer rate) + Controller overHead

Each I/O component has its own controller that add time

### FCFS
The simplest scheduler is **First Come First Served** which is fair but not always fastest. To better understand: if some data request come as third with the first two located far from each other but the last one it's located near the first request, it may be faster to server the first and last and only then the second.

### SCAN
The SCAN Algorithm position the arm in one end of the disk and move to the other end serving in position of data, not time requested.
This may cause latency if some request happens near the start when the algorithm already started. This means that it needs to wait for the arm to arrive at the end and then come back to the start

### C-SCAN
Like the scan but it does not respond to request when coming back to the start. It start from 0 to the end and at this point return to the start without listening to any request while coming back (The SCAN will respond even when coming back)

### Selecting Scheduling Algorithm
FCFS is a common one
SCAN and C-SCAN perform better for system with heavy load on the disk

#Linux implement a deadline:
- 2 separate queues for read and write:
	- Priority for the reads
#### Calculating performance
4KB block
7200 RPM disk
5ms avg seek time
1Gb/s transfer rate
0.1ms controller overhead

5ms + 4.17ms + 0.1ms + transfer timer = avg 
constant time = 5ms + 4.17ms + 0.1ms = 9.27ms
**need to find the time to transfer the variable amount of data**
7200 rotation in one minute => seek time 1minute(60 seconds) / 7200 = 0.008333 s => 8.333ms 

Time per rotation = 60 s / 7200 RP = 8.333ms (seek time)
4KB = 4\*8\*1024=32768bit =  0.032768Gb 
Transfer time = (1Gb/s ) / 0.032768 = 0.031ms

Avg I/O time for 4KB block = 9.27ms + 0.031ms = 9.301ms

## Nonvolatile Memory Devices
NVM devices are electrical rather than mechanical, thus eliminating the mechanical failure. On the other hand more electronical failure may happens 
> radiation in airplane may flip a bit since the data is stored as energy and radiation in this context are just big burst of energy that can easily alter the bit stored. It's important to note that although possible the electromagnetic wave need to hit the right particle with the right amount of force, thus is more probable in airplane where there are more radiation but highly unlikely (not impossible!) at sea level height.

NVM is used in a disk-drive-like container called solid-state disk **SSD**

Challenges:
- Read and written in "page" increments (like the sector of an HDD) but cannot be overwritten in places:
	- The NAND cells have to be erased first
	-  The erasure which occurs in a block increment that is several pages in size, take much more time than a read or write. 
	- Can only be erased a limited number of times before worn out (~100,000) 
	- Life span measured in Drive Writes Per Day (DWPD) that measures how many times the driver capacity can be written per day before it fails. 
		- A 1TB NAND drive with a rating of 5DWPD is expected to have 5TB per day written within the warrantee period without failing

NAND Flash Controller Algorithms
Some pages are marked as invalid. This is done instead of overwriting, the data is transfered into another sector with the updated data. When there are no more valid pages, the **garbage collector** erase all the data inside of them and mark them as valid to make more space available

The controller maintains a FTL (flash translation layer) table to track which pages are valid
![[Pasted image 20240513140147.png]]
## Volatile Memory
DRAM frequently used as mass-storage device
This is done for data that do not need permanent storage location but just faster access time. For this reason the RAM is used instead of external drive.

## Disk Attachment
Different technology used to connect external storage devices to the system bus or I/O bus.
In the modern day the **SATA**  (Serial Advanced Technology Attachment) connector is the most popular. But it lack some speed for the new NVM devices.
For this reason the **NVMe** (NVM express) connector was created that connect directly into the system PCI bus

# Error Detection and Correction
Error detection determines if a problems has occurred (like a bit flipped)
- If detect a bit in DRAM changed from 0 to a 1 halt the operation before the error is propagated.
- Usually done using parity bit (number of bit set to 1 or 0 is even or odd)

Different type of checksum to detect error and something it can be used also to correct it  (single parity bit not sufficient)
To correct it can be used CRC (Cyclic Redundant Code) or ECC (Error Correction Code)

# RAID Structure
Storage continues to get smaller and cheaper, thus it become very possible to use multiple disk in parallel to allow redundancy or better performance

Redundant arrays of independent disks (RAIDs) is used to have redundancy and thus creating a more effective error detection

Instead of a single HDDs of 1TB use three of them in parallel at the same time. The end user will see 1TB but each bit is duplicated in all of the HDDs making it very reliable if a bit is flipped as the others 2 will have the right bit and thus making the consensus to fix the broken one

In a system of 2 disk with full parallelization
Data loss means that both disk need to fail to completely loss some data. When one disk fails just change the broken one and the data will be restored. 
- **Mean Time To Failure**: when a disk fails
- **Mean Time To Repair**: when the disk need to be replaced.
If a disk is lost when the repair is done a failure happens as the data is not yet completely redundant and some data loss happened. This is the **Mean Time To Data Loss**

## Calculate Probability
MTTF = 100,000 hours
MTTR = 10 hours
Probability of failure of one disk = 1 / MTTF = 10^-5
MTTDL = 1 / Probability of data loss (DL)

$PDL = (P(f_1)+P(f_2)) * (p(f_1 or f_2) * MTTR) = (10^-5 + 10^-5) * (10^-5 ** 10) = 2 * 10^-9$

MTTDL = 1/2 \* 10^-9 = 5 10^8 hours

## RAID for improvement in performance
Instead of redundancy the data can be send in parallel to multiple disk, thus improving the throughput.

If need to store 8 bit, and there are a raid of 8 disk with a **Bit-level stripping**: just need the time to store one bit and in parallel it would be done faster than sending 8 bit in a single disk

Another strategy is the **Block-level stripping** and thus a block *i* of a file goes to the (i mod n) + 1 where n is the number of the disk

## RAID Level
There are multiple possible configuration for the RAID to have a good balance from performance (best RAID0) to redundancy (with different option to choose from)
![[Pasted image 20240513142748.png]]


# I/O Systems
I/O management is a major concern of operating system design and operation
It's one of the most important aspect used by the end user (mouse, keyboard, storing files, etc.)

The OS uses **device driver** to talk to the I/O. This let's the connection modular and leave the oddities of each individual hardware to the manufacturer that just need to create the correct software to let the OS talk to their devices.

The structure is as follows:
- Operating System:
	- Has a **device driver** module to encapsulate the connection to the external connection of a device
- Device:
	- *Controller* that connect and communicate to the **device driver**

The controller has different registers for data and control signal that allows the communication to happens.

Devices have addresses:
- Direct communication: direct I/O instructions that transfer data to an I/O port address 
- Memory-Mapped (Much more common): Device control register are mapped into cpu address space. CPU execute I/O request using the standard data transfer using the right address to send the data into the device register

## Polling
Assuming 2 bits for coordination between controller and host

for each byte:
- the host reads repeatedly the busy bit until that bit become clear (0 means ready)
- host sets **write bit** and store a byte into the **data-out register**
- host set **command-ready** bit
- When the controller notices the **command-ready** bit, it sets the **busy** bit
- Controller read **write bit** and thus read from the data-out register to retrieve the information from the host register, then does its operation with the data retrieved
- Controller clears **command-ready** bit, clears the error bit in the status register to indicate that the device I/O succeeded, and clears the busy bit to indicate that it is finished.

The first step is done in a loop until the busy bit becomes clear

It's very inefficient!

## Interrupt
CPU has a pin called Interrupt request. The controller send an Interrupt Request to the CPU and thus the host will jump to the **interrupt handler** to fulfill the request. There is an interrupt vector with different address where to find the instruction needed to fulfill the request and thus the controller will need to send a request with the relative address of the handler (the same operation cannot be done both for HDDs operation and a mouse!). 

**Exceptions** occurs within the processor like arithmetic overflow or segmentation fault.

**Traps**: raised by software and the relative handler will manage that particular situation. This can be used when requesting reading or writing files.

## Direct Memory Access
It's used to avoid programmed I/O for large data movement. This bypass the CPU but needs a DMA controller

DMA has a mini processor to handle small task for data transfer. This will let the CPU to avoid stalling waiting for the I/O offloading the operation to the DMA that can wait for it.
Much more efficient and fast but at cost of more component in the system.

## Application I/O Interface
I/O system call let's the application interact with the external devices encapsulating the device behaviors in generic classes

Different type of devices:
- Character stream or block
- Sequential or random-access
- Synchronous, asynchronous or both
- Sharable or dedicated
	- Usable concurrently or not
- speed of operation
- Read-write, read only, write only


# File
#2024-05-20
Other than the byte stored inside the file, it has others information (readable, writable, ownership and others), those **file attributes** are also called metadata of the file.
## File Attributes
- name
- identifier
- type
- location
- size
- protection
- time, date, user identities

## Open File
An opened file should be protected based on the situation. Multiple write will cause overwrite of the information stored by one of the two process. On the other hands multiple process can read it without causing any problems (although it should stop writing operation to avoid problems).

### File Locking
To resolve those issue the file locking is used in its different forms:
- Shared lock: multiple process can acquire the lock concurrently (multiple read operation)
- Exclusive lock: only one process can acquire such lock (write operation, no read/write operation allowed outside the of the owner of the lock)

Although this kind of lock exist their mechanisms may be applied differently:
- Mandatory: once a process acquire a lock the OS will prohibit any other acquisition. The OS ensure the lock is properly acquired and released.
- Advisory: It is up to the software developer to ensure the lock is properly acquired and released. 

Windows: mandatory
Linux: advisory

## File Types - Name, Extension

| file type      | usual extension       | function                                                                            |
| -------------- | --------------------- | ----------------------------------------------------------------------------------- |
| executable     | exe, com, bin or none | ready-to-run machine language program                                               |
| object         | obj, o                | compiled machine language, not linked                                               |
| source code    | c, cc, java, asm...   | source code in various language                                                     |
| batch          | bat, sh               | commands to the command interpreter                                                 |
| text           | txt, doc              | textual data                                                                        |
| word processor | wp, tex, rtf, doc     | various word-processor formats                                                      |
| library        | lib, a, so, dll       | libraries of routines for programmers                                               |
| print or view  | ps, pdf, jpg          | ASCII or binary file in a format for printing or viewing                            |
| archive        | arc, zip, tar         | related files grouped into one file, sometimes compressed, for archiving or storage |
| multimedia     | mpeg, mov, mp3, avi   | binary file containing audio or A/V information                                     |

## Directory Structure
A collection of nodes containing information about files
> both directory structure and files reside on disk

Disk can be subdivided into partitions (can be **RAID** protected against failure) and can be **raw** without a file system or **formatted** with a file system

The partition are also called minidisks, slices.
The entity the contains the file system is known as volume. Each volume also track the system's info in **device directory** of **volume table of content**.

File system can be **general-purpose** or **special-purpose**.

And a typical file system organization can be as follows
![[Pasted image 20240520135932.png]]

# Memory allocation
#2024-05-27
## Contiguous Allocation
Starting from a block in memory continue allocating in succession
Possible external fragmentation if file need to increase in size and no free block available in successive memory position, this is a problem

### Extend-Based Systems
Solution of contiguous allocation unavailability of free space
- determining how much space is needed for a file
- contiguous chunk of space is allocated initially
- if more storage is needed, another contiguous chunk is allocated in another part of memory: this is called **extent**
- Location of file's block is recorded as location and block count plus a link of the first block of the next chunk 

## Linked Allocation
Each block record the position of the next block (similar to a linked list)
The last block will have the **EOF** to stop reading the next one

If the block #3  is needed, where will the OS will find it? It's unknown until all the previous block are read to find the required one

Pros:
- No external fragmentation
- Easily expandable if file need to grow in size:
	- Just find the first free block and place it while recording its position in the previous block

### File Allocation Table - #FAT
An important variation  on linked allocation is the use of a **File Allocation Table**
- Section of storage at the beginning of each volume is set aside to contain the table.
- The table has one entry for each block
- The directory entry contains the block number of the first block of the files.
- Table entry indexed b that block number contains the block number of the next block in the file
- The chain continues until it reached the last block which has a special end-of-file value as the table entry

## Index Allocation
Linked allocation solves the external-fragmentation and size-declaration but in absence of **FAT** linked allocation cannot support efficient direct access

**Index allocation** solve this problem by creating an **index block** to point all the block memory address

The allocation occurs like the linked method, but the single block does not contain the index of the next block, instead a new block is allocated just for this. 
The directory, thus, will contain the address of the **index block** from which all the others block can be found.
$i^{th}$ index will point to the $i^{th}$ block of the file
cons:
- with small file the index block is very inefficient
## Combined Scheme: UNIX UFS
Combine both linked and index allocation:
- Small file:
	- Use linked allocation
- Large file:
	- Use index allocation

## Free-space management
OS have a free-space list of block that can be occupied
The list can be implemented both as **bitmap** or **bit vector**
Each block is represented by 1 bit
- if bit is 
	- 1: the block can be allocated
	- 0: block occupied

- Grouping:
	- Modify linked list to store address of next n-1 free block in first free block
- Counting:
	- space is frequently contiguously used and freed, with contiguous-allocation, extents or clustering
		- keep address of first free block
		- free space list then has entries containing addresses and counts

The counting use the fact that a file will use multiple block in contiguous matter and usually also if it free, the whole file block is freed
