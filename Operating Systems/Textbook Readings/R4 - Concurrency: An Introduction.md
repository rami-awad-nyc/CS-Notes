## Concurrency

### Threads

In a multi-threaded program, each *thread* is like a separate process except
that all threads share the same address space.  

If two threads are running on a single processor, a *context switch* must take
place to switch from T1 to T2 (similar to processes). Before T2 can run, register
state of T1 must be saved and register state of T2 must be restored.  
- With processes, state is saved to a process control block (PCB).  
- With threads, state is saved to a thread control block (TCB).

In a multi-threaded process, there are multiple stacks at the top of memory, one
for each thread. Example:   
```
Stack(1)
--free
Stack(2)
--free
--free
Heap
Program Code
```

### Why Use Threads?

Two major reasons:  
1. Parallelism: Using one thread per processor is more efficient than using only
one processor. Turning a single-threaded program into a multi-threaded one is
called *parallelization*.  
2. I/O: With threads, a program can avoid getting stuck while waiting on I/O.
AKA threading enables overlap of I/O with other activities within a single
program. Why not just use multiple processes? It is easier to share data with
threads. Processes are better when less data needs to be shared.

### Thread Creation

In C:  
- `pthread_create()` used to create a thread (executes a function).  
- `pthread_join()` waits for a particular thread to complete.

When two threads run on a processor, the OS *scheduler* decides which thread
executes at any time. It is difficult to estimate which instruction the
scheduler will choose. In other words, the output is not *deterministic*.   
- Lines of C may be multiple lines of assembly instructions. The scheduler
determines which assembly instructions execute, not which lines of C. Use a
disassembler to see these instructions: `objdump -d executable_file` on linux.
Takeaway: BREAK DOWN CODE INTO ASSEMBLY TO DETERMINE POSSIBLE OUTPUTS.   

Since threads can access shared data, concurrency can become a severe issue.
*Race conditions* occur when results depend on timing of code's execution. If a
context switch occurs at an untimely point in execution, we get the wrong
result.   
- When multiple threads execute code that can result in a race condition, the
  code is considered a *critical section*. In other words, a critical section is
  a piece of code that accesses a shared variable and must not be accessed by
  multiple threads at once.

### Mutual Exclusion

To prevent race conditions, the program needs a *mutual exclusion*, which
guarantees that if one thread is executing in a critical section, other
threads won't.  
- *Atomicity* is turning a section of code into one block that either executes
  from start to finish or doesn't with no interruptions in between.  
- *Synchronization primitives* needed to treat critical sections and access them
  in a synchronized and controlled manner.  


