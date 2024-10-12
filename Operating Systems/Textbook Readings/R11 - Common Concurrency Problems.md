## Common Concurrency Problems

Bugs can be categorized as either non-deadlock bugs or deadlock bugs.

### Non-Deadlock Bugs

Two major types of non-deadlock bugs: atomicity-violation bugs and
order-violation bugs.

Atomicity-Violation Bugs:  
- A code region is intended to be atomic, but the atomicity is not enforced
  during execution. 
- The solution is straightforward: surround critical sections  with locks.

Order-Violation Bugs:  
- Thread A should be executed before thread B but the order is not enforced
  during execution.  
- The solution is straightforward: use condition variables to enforce ordering.

### Deadlock Bugs

Deadlock occurs when, for example, Thread 1 holds Lock 1 and waits for Lock 2.
Thread 2 holds Lock 2 and waits for Lock 1. Both the threads will wait forever
as neither can acquire the lock the other thread has in order to release their
current lock.

Four conditions need to hold for a deadlock to occur:  
- Mutual exlcusion: threads claim exclusive control of resources that they
  require (thread grabs a lock)  
- Hold-and-wait: Threads hold resources allocated to them while waiting for
  additional resources (thread holds locks while waiting for other locks).
- No preemption: Resources (locks) cannot forcibly be removed from threads that
  are holding them
- Circular wait: there exists a circular chain of threads such that each thread
  holds one or more resources that are being requesting by the next thread in
  the chain.

If any of these conditions are not met, deadlock cannot occur.

Preventing circular wait:  
- The most practical solution is to write lock code such that you never induce
  circular wait. 
- The easiest way to do this is to provide a *total ordering* on
  the lock acquisition. For example, if there are two locks in the system, you
  can prevent deadlock by always acquiring Lock 1 before Lock 2.
- In complex systems with many locks, this can be difficult. A *partial
  ordering* works as well.
- Ordering requires careful design of locking strategies. It is also a
  convention and cannot be enforced. It is helpful to leave comments to remind
  the programmer any ordering conventions.

Preventing hold-and-wait:  
- This requirement can be avoided by having a thread acquire all locks at once.
  First, it needs an additional lock typically called `prevention` that
  surrounds the acquisition of all the locks.
- This solution is problematic because we need to know which locks must be held to
  acquire them ahead of time. Also, this decreases concurrency as locks are
  acquired early on rather than when they are needed.  

Preventing no premeption:  
- Thread libraries provide more flexible sets of interfaces to help avoid
  deadlocks. Specifically, the routine `mutex_trylock()` either grabs the if
  available or returns an error if the lock is held. If the lock is held, you
  can try again.
- Let's say a thread acquires Lock 1. It then calls `trylock` on Lock 2. If an
  error is received, release Lock 1 and go to the very top. If there is a
  deadlock, the other thread is able to acquire Lock 1 from the current thread.
- If two threads repeatedly attempt this sequence and fail to acquire the other
  thread's locks, they are stuck in a *livelock*, which is not the same as a
  deadlock. To fix this, add a random delay before looping to the top again.
- Cons? The jump back to the top could be costly. Also, if other resources were
  acquired along the way, they must be safely released as well.

Preventing mutual exclusion:  
- This is difficult because the code has critical sections. One idea behind a
  *lock-free* approach is to use hardware instructions to build data strcutres
  without locking. 
- In a *lock-free* approach, use the `CompareAndSwap` hardware instruction
  rather than locks. Surround critical section with a do-while loop:  
  ```
  do {
      // critical section
  } while (CompareAndSwap(&variable, old, new)==0);
  ```   
- Of course, this is more difficult with more complex code.

Deadlock Avoidance via Scheduling:  
- Avoidance requires some global knowledge of which locks various threads might
  grab during execution and scheduling the threads to guarantee no deadlock can
  occur. 
- For example, if you have 2 CPUs and 4 threads and if you know that no deadlock
  can occur as long as Thread 1 and Thread 3 aren't run at the same time, you
  would enable the scheduler to run Thread 1 and Thread 2 so that Thread 3 and
  Thread 4 may run at the same time. 
- However, there are no guarantees that deadlock cannot occur and sometimes the
  scheduling process here could cause job lengths to increase significantly.
  Because of this, deadlock avoidance is rarely used.

Detect and Recover:  
- The final strategy is to let deadlocks occasionally occur and then take
  action. For example, if the OS freezes once a year, just reboot it.
