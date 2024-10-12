## Semaphores

Semaphores are objects with integer values that can be manipulated with two
routines:`sem_wait()` and `sem_post()`. Semaphores replace locks and condition
variables. To use a semaphore, we must first initialize it:  
```
#include <semaphore.h>
sem_t s;		// declare semaphore object
sem_init(&s, 0, 1);	// assign 1 to sem.
			// The 0 means that all threads in the process share the
			// sem
```

`sem_wait(&s)`:
- This function decrements the value of the semaphore by 1 and then waits if the
  value of the semaphore is negative

`sem_post(&s)`:  
- This function increments the value of the semaphore by 1 and then wakes a
  thread if there are more than one threads waiting.

The value of the semaphore, when negative, is equal to the number of waiting
threads. 

### Binary Semaphores (Locks)

Semaphores can be used as locks. Surround critical sections with `sem_wait()`
and `sem_post()`. The initial value of the semaphore must be 1. 

Example to make understanding easier:  
Thread 1 calls `sem_wait()` and enters critical section. The semaphore value
decreases to 0. Thread 2 calls `sem_wait()`. The value decreases to -1. Now
thread 2 waits until thread 1 calls `sem_post()` because the semaphore value is
negative. Also note that there is one thread waiting and the value is -1. 

### Semaphores For Ordering

Semaphores can be used an ordering primative (similar to condition variables).
We often find one thread waiting for something to happen and another thread
making that something happen and then signalling that it has happened. In this
case, use `sem_wait()` to put a thread to sleep until `sem_post()` is called by
another thread. Make to sure to initialize value of the semaphore to 0.


### Rule Of Thumb

Initialize a semaphore to the number of resources willing to give away
immediately after initialization. With lock, it is 1 because you are willing to
have the lock given away immediately. With cond variables, it is 0 because you
aren't giving anything away. Rather, you are waiting until a signal is received.

### Producer/Consumer (Bounded Buffer) Problem

Three semaphores are used: empty (initialized to a value equal to the max number
of items in the buffer, like 10 for example), full (initialized to 0), and mutex
(initialized to 1 -- binary semaphore). 

The producer keeps adding data until the buffer has no empty items and then it
waits (`sem_wait(&empty)`). After adding an item, the producer calls
`sem_post(&full)` to indicate that there is now a filled item for the consumer
to consume. The consumer keeps consuming data until the buffer has no filled
items and then it waits (`sem_wait(&full)`). After consuming an item, the
consumer calls `sem_post(&empty)` to indicate that there is now an empty item
that the producer can fill in. 

Lastly, surround critical sections with `sem_wait(&mutex)` and
`sem_post(&mutex)` to represent a lock. To avoid deadlock, make sure that the
lock wait and post are inside of the condition variable wait and post.

### Reader-Writer Lock

Imagine a linked list with two operations: insert and lookup. If inserting, a
lock is of course needed to prevent shared data from being accessed by many
threads. If reading data, all we need to do is guarantee that no inserts are
currently happening. This type of lock is called a *reader-writer* lock.

When a thread wants to insert data, a writer lock is needed (can be binary
semaphore of course). This is straightforward. Simply surround insert operations
with lock to prevent multiple threads from writing at the same time.

When threads want to read data, must ensure that no thread is currently writing
data. Basically, the first reader should acquire both the read lock and the
write lock. Now, no thread can write data as long as there is a reader. After
this first reader has acquired the read lock, other threads can also acquire the
read lock since no data is being modified.

```
acquire_writelock(): sem_wait(&writelock)
release_writelock(): sem_post(&writelock);
acquire_readlock(): sem_wait(&readlock); reader_count++; if (readers==1){
			sem_wait(&writelock); } sem_post(&readlock);
release_readlock(): sem_wait(&readlock); reader_count--; if (readers==0){
			sem_post(&writelock); } sem_post(&readlock);
```

Note that `acquire_readlock()` acquires and then releases readlock to let other
threads read data without getting blocked. Readlock only exists to protect the
`reader_count` and the writelock from being modified by many threads. In
comparison, `acquire_writelock()` acquires but doesn't release writelock.

### The Dining Philosophers

There are 5 philosophers and a fork between each pair of philosophers. Each
philosopher calls `think()`, which doesn't need any forks, and then `eat()`,
which requires both their left fork and right fork. They then place these forks
down and repeat.  
```
while (1) {
    think();
    get_forks(p);
    eat();
    put_forks(p);
}
```   
How to implement `get_forks` and `put_forks` so that no philosopher starves and
concurrency is maximized? Forks and philosophers range from 0 to 4. Also, 5
semaphores are used, one for each fork.

If we acquire a lock for the fork on the left first and then the right for each
philosopher, then we reach a deadlock. Each philosopher will be stuck holding a
fork and waiting for another forever. Solution? One of the philosophers must do
something different. If philosopher 4 grabs the right fork instead of the left, the
deadlock is broken. 

Similar problems include the smoker's problem and the sleeping barber problem.

### Thread Throttling

How to prevent too many threads from doing something at once and bogging the
system down? Use a semaphore to limit the number of threads concurrently
executing the piece of code. This is called *throttling*.

### Implementing Semaphores

You can implement a basic semaphore using mutex and condition variables fairly
easily, although matching Dijkstra's semaphore is significantly more complicated.

A semaphore struct has a value, a mutex, and a condition variable. When running
`wait()`, lock the mutex and then condition wait for the value to stop being
negative. Once the thread wakes up (and value >= 0), then subtract value by 1.
Finally, release the mutex.

When running `post()`, lock the mutex and then increment the semaphore value.
Then, signal the sleeping thread to wake up. Lastly, release the mutex.

