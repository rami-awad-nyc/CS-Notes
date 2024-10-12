## Coding Standards for Programming with Threads

1. Always do things the same way, even if one way is not better than another. In
other words, follow the same strategy every time so that you can focus on the
core problem and it is easier to read the code.  

2. Always use monitors (conditional variables + locks). Either semaphores or
monitors can be used to write concurrent programs, but monitors are
significantly more clear and pretty much self-document the code. In C++,
monitors are a convention rather than a built-in feature.  

3. Always hold lock when waiting or signalling a condition variable.

4. Always grab lock at beginning of procedure and release right before return.
If you find yourself needing to grab a lock in the middle of a procedure, that
is usually a red flag that you should break the piece you are considering into a
separate procedure. 

5. Always use while loops rather than if statements when checking a condition
before calling `cond_wait()`. 

6. (Almost) never use `sleep()` to wait for another thread to do something. The
correct way is to use `wait()` on a condition variable.

### Problem Solving Strategy

1. Decompose problem into objects  
- Identify units of concurrency. Make each a thread.
- Identify shared chunks of state. Make each shared thing an object. Identify
  the high-level actions made by threads on these objects.
- Write down the high-level main loop of each thread.

For each object:  

2. Write down the synchronization constraints on the solution. Identify the type
of each constraint: mutual exclusion or scheduling. 

3. Create a lock or condition variable corresponding to each constraint.

4. Write the methods, using locks and condition variables for coordination.
