## Interlude: Thread API

These notes are for C using the POSIX thread library.  
POSIX is a set of standards that ensures compatibility between different
operating systems. It mainly focus on APIs, like the POSIX thread library in
C.

All pthread functions return an error code. Always make sure to assert that the
error code is 0. For simplicity, this is not included in these notes.

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
  * Otherwise, make sure to cast argument as `(void*)` type

To wait for a thread to complete, call the function `pthread_join(pthread_t
thread, void **value_ptr)`.  
- The first argument is the thread to wait for  
- The second argument is a pointer to the return value expected to receive. If
  we don't care about return value, just pass NULL instead.  

If returning a pointer to memory on the heap, do it like this: `return (void*) memory;`

Example code in `main`:  
```
pthread_t p;					// declare new thread
struct myarg args = {10, 20};			// define own struct for arguments
int returnVal;					// store return value
pthread_create(&p, NULL, function, &args); 	// create new thread 
pthread_join(p, &returnVal); 			// store return value in returnVal
```

### Locks

Mutual exclusion to a critical section is done using *locks*.  

Locks must be initialized. Two ways:  
- At compile time: `pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;`  
- At run time:  
	```
	pthread_mutex_t lock;
	pthread_mutex_init(&lock, NULL); 
	```   
  * The first argument is the address of the lock, which must be declared.   
  * The second argument is set of attributes, NULL uses the defaults.   
- When finished with lock, call `pthread_mutex_destroy(&lock);`   

POSIX thread mutex:   
```
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER; 	// initialize lock
pthread_mutex_lock(&lock); 				// lock the mutex
... /* critical section */ 				// critical section
pthread_mutex_unlock(&lock); 				// unlock the mutex
pthread_mutex_destroy(&lock); 				// destroy lock
```

If one thread acquires a lock, other threads will wait for release (get blocked).  

### Condition Variables

*Condition variables* used when some kind of signaling must take place between
threads.  

Two primary routines used for conditional variables:  
- `pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);`
  * Puts calling thread to sleep and waits for another thread to signal it  
  * Also releases lock until woken, when it acquires lock again
  * First argument is address of conditional variable  
  * Second argument is address of lock  
- `pthread_cond_signal(pthread_cond_t *cond);`  
  * Signals sleeping thread to wake up  
  * Argument is address of conditional variable  

Use a while loop to check the condition rather than an if statement. Also it is
tempting to only use a regular variable to signal between threads, but DON'T.
You must use both a regular variable and a condition variable.

POSIX thread cond variable:  
Thread 1: 
```
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER; 	// initialize lock
pthread_cond_t cond = PTHREAD_COND_INITIALIZER; 	// initialize CV
pthread_mutex_lock(&lock); 				// lock mutex
while (ready == 0) 					// sleep until update
	pthread_cond_wait(&cond, &lock); 		// and signal 
pthread_mutex_unlock(&lock); 				// unlock mutex
pthread_cond_destroy(&cond);				// destroy cond variable 
```
Thread 2:  
```  
pthread_mutex_lock(&lock); 	// lock mutex
ready = 1;			// change variable
pthread_cond_signal(&cond); 	// after var change, THEN signal
pthread_mutex_unlock(&lock); 	// unlock mutex
```
