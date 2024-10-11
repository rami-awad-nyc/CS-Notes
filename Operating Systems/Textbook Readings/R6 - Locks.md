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

### Spin Locks

Loads and Stores: a struct has a variable called `flag` that is either 0 or 1. When `lock(&mutex)` is called, a while loop does nothing as long as
`flag==1`. If `flag` ever equals 0, then set `flag` to 1. When `unlock(&mutex)`
is called, set `flag` to 0.  
- Why doesn't this work? First, there is no mutual exclusion. If both threads
  escape the while loop before setting `flag` to 1, then both threads will set
  `flag` to 1 afterwards, meaning they both acquire the lock, which shouldn't be
  possible. Also, if a lock is acquired, then another thread will endlessly
  check the value of the flag (*spin waiting*). This has bad performance.  

Test-And-Set Spinlock: to implement a spinlock properly, a little hardware
support is necessary. A *test-and-set* or *atomic exchange* instruction.  
- Test-and-set is an atomic or single instruction that takes in a memory address
  and a value. It goes to the memory address and replaces the data with the new
  value. Then it returns the old value.  
- Basic Spinlock:  
```
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *lock) {
    // 0: lock is available, 1: lock is held
    lock->flag = 0;
}

 void lock(lock_t *lock) {
     while (TestAndSet(&lock->flag, 1) == 1)
     ; // spin-wait (do nothing)
 }

void unlock(lock_t *lock) {
    lock->flag = 0;
}
```
- Spinlocks require a *premeptive scheduler*, one that interrupts threads, if
  used on a single processor.  
- Evaluation: Mutex? Yes. Fairness? No. Performance? If number of threads =
  number of CPUs, yes. Otherwise, no.

Compare-And-Swap:  
- Another hardware primitive (single instruction) called *compare-and-swap* or
  *compare-and-exchange*. Take in 3 inputs, a memory address and 2 values.
  Compare the value at the memory address with the first value. If they are the
  same, then replace the data at the memory address with the second value.
  Return the original value.    
- Identical when replacing test-and-set in the basic spinlock above, but has
  more powerful applications.

Fetch-And-Add:  
- Another hardware primitive called *fetch-and-add*. It takes in a memory
  address, finds the data at that address and increments it by 1. Then, it
  returns the old value.   
- Application? Ticket lock

If a thread gets interrupted before releasing a lock, the next thread will spin
until the CPU is returned to first thread. Solution? Yielding.  
- The OS provides an OS primitive `yield()`, which a thread can call when it wants
  to give up the CPU. Threads can be in three states: *running, ready, blocked*.
  Yield moves a thread's state from running to ready and then promotes another
  to running.  
- In the spin-waiting loop, `yield()` instead of doing nothing.  
- Cons: still costly in terms of context-switching. Also, doesn't address
  starvation.

**Final solution** that accounts for context-switching and starvation: using a
queue to sleep instead of spin. If thread tries to acquire lock but it is
already held, add the thread to a queue using its ID. Then put the thread to
sleep using an OS call (varies by system). Linux provides `futex` calls.
