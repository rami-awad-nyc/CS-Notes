## Scheduling: The Multi-Level Feedback Queue

*Multi-level Feedback Queue (MLFQ)* wants to optimize turnaround time and
minimize response time, but how (considering the OS doesn't know how long a
process will take to complete)?

### MLFQ: Basic Rules

The MLFQ has a number of distinct queues, each assigned a different priority
level. At any time, a job that is ready to run is on a single queue. MLFQ
decides which jobs to run based on priority (a job on the highest queue runs
first). If two jobs have the same queue, then RR scheduling is used.

Rule 1: If Priority(A) > Priority(B), A runs  
Rule 2: If Priority(A) = Priority(B), A & B run in RR

MLFQ varies priority of a job based on its observed behavior. The MLFQ learns
about the process as it runs. 

The priority of a process should change over time, or else some jobs might get
starved.

### Attempt 1: How to Change Priority

A job's *allotment* is the amount of time a job can spend at a given priority
level before the scheduler reduces its priority. For simplicity, this is the
same amount of time as a time slice.

Rule 3: When a job enters the system, it is placed at highest priority  
Rule 4a: If a job uses up its allotment, it moves down a queue  
Rule 4b: If a job gives up the CPU before allotment is up (I/O operation for
example), then it stays at the same priority level (allotment reset).

Problem? If there are too many interactive jobs, they will combine to consume
all CPU time and long running jobs starve. Also, clever users can trick the
scheduler to give them more than a fair share of resources.

### Attempt 2: The Priority Boost

Rule 5: After some time period *S*, move all the jobs in the system to the
topmost queue. This solves starvation and CPU-bound jobs becoming interactive.

What should S be set to? If too high, long-running jobs could starve. Too short
and interactive jobs don't get proper share of CPU.

### Attempt 3: Better Accounting

Rules 4a and 4b can cause gaming of the scheduler. To fix this, keep track of
allotment even if I/O occurs. Whereas before if a process was interrupted, then
its allotment would be reset. However, instead, we should store its value and
continue where it left off when the process later continues running.

Rule 4: Once a job uses up its time allotment at a given level (regardless of
how many times it has given up the CPU), its priority is reduced (it moves down
a queue).

### Summary

Rule 1: If Priority(A) > Priority(B), A runs and B doesn't  
Rule 2: If Priority(A) = Priority(B), A and B run in RR  
Rule 3: When a job enters the system, it is placed at the highest priority (the
topmost queue)  
Rule 4: Once a job uses up its time allotment at a given level (regardless of
how many times it has given up the CPU), its priority is reduced (moves down a
queue)  
Rule 5: After some time period *S*, move all the jobs in the system to the
topmost queue  


