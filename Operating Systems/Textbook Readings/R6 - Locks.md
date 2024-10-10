## Locks

Programmers put locks around critical sections to ensure these sections are
treated as a single atomic instruciton.

Locks are implemented in C using the pthread library, but for simplicity, these
notes use abstract functions not belonging to any library.

### Lock Basics

A lock is just a variable that must be declared. A lock variable is either
*available* (aka *unlocked* or *free*) and thus no thread holds the lock or
*acquired* (aka *locked* or *held*) and thus one thread holds the hold and is in
a critical section.

If a lock gets acquired by a thread, other threads must wait to acquire it. If
no threads are waiting, the state of the lock is changed back to free. 

Instead of one big lock that is used any time any critical section is accessed
(a *coarse-grained* locking strategy), one usually protects different data with
different locks (a *fine-grained* locking strategy).

### Evaluating Locks

How to implement your own lock? First, necessary to evaluate a good lock.

Before building locks, criteria must be established:  
- Lock provides *mutual exclusion*, preventing multiple threads from entering a
  critical section.  
- Lock has *fairness*: each thread contending for the lock gets a fair shot at
  acquiring it once it is free. No threads should *starve* (get blocked
  forever).  
- Lock's performance isn't affected by too many overheads. No concerns when
  threads are contending for lock on single CPU and multiple CPUs.

### Controlling Interrupts

One of the first solutions to provide mutual exclusion was disabling interrupts for
critical sections for single processor. `lock()` disables interrupts and
`unlock()` reenables interrupts.  
- Cons:  
  * Threads perform a *privileged operation* and this can be abused: a program
    calls `lock()` and goes into endless loop. OS will never regain control of
    system.
  * Does not work on multiple processors. Even if interrupts are disabled on one
    CPU, threads can still run on other processors.  
  * Interrupts being disabled for too long can lead to interrupts being lost.


