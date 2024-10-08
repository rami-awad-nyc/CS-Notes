## Interlude: Thread API

These notes are for C using the POSIX thread library.  
POSIX is a set of standards that ensures compatibility between different
operating systems. It mainly focus on APIs, like the POSIX thread library in
C.

### Thread Creation and Completion

To create a thread, call the function: `pthread_create(pthread_t *thread,
const pthread_attr_t *attr, void *(*start_routine)(void*), void *arg)`.  
- The first argument `thread` is a pointer to a thread of type `pthread_t`  
- The second argument `attr` is for setting attributes this thread might have
  like stack size. Usually, we just pass `NULL`.  
- The third argument is a *function pointer* and determines which function this
  thread should start running in. Simply type the name of the function, no
  parantheses.  
- The fourth argument is the argument to be passed into the function.  
  * If multiple arguments are needed, then package them into a single type.

To wait for a thread to complete, call the function `pthread_join(pthread_t
thread, void **value_ptr)`.  
- The first argument is the thread to wait for  
- The second argument is a pointer to the return value expected to receive. If
  we don't care about return value, just pass NULL instead.  

Be careful not to return a pointer which refers to something allocated on the
thread's call stack because this memory will be deallocated when the thread
completes.  
- If returning malloc memory, do it like this: `return (void *) memory;`

### Locks

Mutual exclusion to a critical section is done using *locks*. 
