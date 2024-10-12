## Monitors (Deprecated)

In monitor classes, the monitor guarantees that only one thread can be active
within the monitor at a time. How? Whenever a thread tries to call a monitor
routine, it tries to acquire the monitor lock. If it succeeds, it will be able
to call into the routine. Otherwise, it'll be blocked.

Why use monitors instead of just explicit locking? At the time of its creation,
people were looking to blend concurrency with OOP. Nothing more than that.

Monitor classes provide implicit use of locks and condition variables in OOP
languages like Java and Python. However, monitors aren't really a thing in C and
C++. You would have to explicitly use locks and condition variables, which is
just regular synchronization.

### Mesa Monitors

Mesa Semantics vs. Hoare Semantics:  
- Hoare Semantics: when a thread signals another thread waiting on a condition
  variable, the signaling thread is immediately suspended and the waiting
  thread runs immediately.  
- Mesa Semantics: when a thread signals another thread waiting on a condition
  variable, the signaling thread state is changed from blocked to ready BUT not
  running. The signaling thread keeps running until the scheduler decides to
  deschedule it. When it eventually gets scheduled, the ready thread re-checks the
  condition with no guarantee that it is true. 

In Mesa semantics, using `broadcast()` to signal all waiting threads always
works because the threads re-check the condition. However, this can lead to
performance issues if only a few of the threads are going to pass the condition
anyway. A *thundering herd* occurs if many threads are woken but only a small
amount actually pass the condition. The rest just get blocked again. 

### Java Monitor

To use monitors in Java, add *synchronized* keyword to the method. A condition
variable is also supplied with each synchronized class. To use it, call `wait()`
or `notify()`. However, note that there is only one condition variable per
method. This prevents functionality for programs like the producer/consumer
problem. You can use `notifyAll()` which is like `broadcast()`, but this can be
inefficient. So, Java later added an explicit Condition class.
