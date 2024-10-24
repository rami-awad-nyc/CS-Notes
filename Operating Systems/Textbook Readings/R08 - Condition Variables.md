## Condition Variables

A thread might wish to check whether a *condition* is true before continuing its
execution.  

### Condition Variable Basics

We could try using a shared variable and a while loop that spins until the
condition is true, but this is hugely inefficient. The alternative is to put the
thread to sleep until the condition becomes true.

To wait for a condition to become true, a thread can use a *condition variable*,
which is a queue that threads put themselves on when some condition is not as
desired. Some other thread, when it changes the condition, can wake one or more
of the waiting threads (*signaling* the condition). 

Condition variables have two basic operations:  
- `wait()` call executed when thread wishes to put itself to sleep  
- `signal()` call executed when thread changes something in program and wants to
  wake up a sleeping thread waiting on a condition

`wait()` takes in a mutex as a parameter and **assumes that it is locked** when
called. Also, always hold the lock when signalling. So hold the lock when
calling `signal()` and `wait()`.

Always use a while loop that checks the condition and contains the call to
`wait()` rather than an if statement.

### Producer/Consumer (Bounded Buffer) Problem

Type of problem where a producer generates data items, putting them in a buffer,
and consumer threads consume them.

Two interpretations of signals:  
- Mesa semantics: signaling a thread wakes the thread up but there is no
  guarantee that the condition has changed  
- Hoare semantics: opposite of Mesa semantics. There is a guarantee that the
  condition has changed.
- Most systems employ Mesa semantics.  

Correct Producer/Consumer Solution: a producer only sleeps if all buffers are
currently filled and a consumer only sleeps if all buffers are currently empty. 

### Covering Conditions

There are certain situations when there are many waiting threads and you don't
know which threads to wake up. In this case, use `broadcast()`, which wakes up
all waiting threads. This guarantees that any thread that should be woken is
woken, but it also means that some threads might wake up, check their
conditions, and go back to sleep (negative performance). This condition is
called a *covering condition*. 
