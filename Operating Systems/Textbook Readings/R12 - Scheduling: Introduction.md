## Scheduling: Introduction

The OS employs scheduling *policies* or *disciplines* to determine which
process/thread runs.

Modern schedulers are *preemptive*, meaning they are willing to stop another
process from running in order to run another. They do this with a *context
switch*, or stopping one process temporarily and resuming or starting another.


### Workload & Metrics

The processes running in the system are sometimes collectively called the
*workload*. We make the following workload assumptions about processes
(also called *jobs*) that are running in the system for this chapter (these are
unrealistic assumptions):  
1. Each job runs for the same amount of time
2. All jobs arrive at the same time
3. Once started, each job runs to completion
4. All jobs only use the CPU (no I/O)
5. The run-time of each job is known

There are many scheduling metrics, but for now, only a single one is used:
*turnaround time*, which is the time it takes to complete a job minus the time
at which the job arrived in the system. For now, time of arrival is 0 since we assume all jobs arrive at the same time.

*Fairness* is measured by *Jain's Fairness Index*. There is often a tradeoff
between fairness and performance (i.e. turnaround time).

### FIFO (First In, First Out)

Also known as *First Come, First Served (FCFS)*. 

Let's say jobs A, B, C arrive at the same time. Each takes 10 seconds to
complete. Let's say A arrives a hair faster than B, and B a hair faster than C.
Then average turnaround time = (10 + 20 + 30) / 3 = 20 seconds.

If we relax assumption 1 and A takes 100 seconds to complete, then average
turnaround time = (100 + 110 + 120) / 3 = 110 seconds. THIS IS VERY LONG.
*Convoy effect* is when a number of relatively short potential consumers of a
resource get queued behind a heavyweight resource consumer.

### SJF (Shortest Job First)

The shortest job runs first rather than the job that arrives first. Now, average
turnaround time = (10 + 20 + 120) / 3 = 50 seconds. Significantly better.
However, what if arrival time was varied (relaxing assumption 2). For example, A
arrives at time = 0 and B,C arrive at time = 10 seconds. In this case, A would
run first, then B, and then C). Average turnaround time = (100 + (110-10) +
(120-10)) / 3 = 103.33 seconds. 

### STCF (Shortest Time-to-Completion First)

STCF is simply SJF with preemption (relaxing assumption 3). It is also called
Preemptive SJF (PSJF). Using the same values from SJF, the average turnaround
time is ((120 - 0) + (20 - 10) + (30 - 10)) / 3 = 50.

### Response Time

If we only job lengths, jobs didn't have I/O, and the only metric was turnaround
time, then STCF is easily the best scheduler. However, we add a new metric
called *response time*, which is the difference between when the job arrives in
a system to the first time it is scheduled. 

In the same example above when STCF is used, response time for A is 0 seconds,
for B is 0 seconds, and for C is 10 seconds. Average response time is 3.33
seconds. This is terrible for I/O (imagine waiting several seconds after
interacting with the system).

### Round Robin

In *Round Robin (RR)* scheduling, instead of running jobs to completion, jobs
are run for a *time slice* or a portion of time (sometimes called a *scheduling
quantum*). RR is sometimes called *time-slicing*. 

Time slices are measured in units of time (like ms, seconds, etc.) and must be
multiples of the timer interrupt, which occurs every few units of time. 

Example: A,B,C each wish to run for 5 seconds and each time slice is 1 second.
Then average response time is (0 + 1 + 2) / 3 = 1.

*Amortization* is used to reduce fixed costs by performing an operation fewer
times. For example, if a time slice is given 10 ms and the context switch takes
1 ms, then the fixed cost is 10% of the total time given. We can reduce this by
giving the time slice 100 ms, so now the fixed cost is only 1% of the total
time given.

If the quantum is too short, then the cost of context switches might dominate
each time slice. If the quantum is too large, then it is as if we are running
FIFO. 

Also, if turnaround time is our metric, then RR is terrible. **Basically, if
turnaround time is the metric (performance), then STCF/SJF is the best, but the
policy lacks fairness. If response time is the metric (fairness), then RR is the
best, but performance is lacking.**

### Incorporating I/O

Now we relax assumption 4. When an I/O request is made, the current job won't be
using the CPU (it is blocked waiting for the I/O completion). 

If A and B take 50 ms to complete and A has an I/O request every 10 ms, then how
can we create an STCF scheduler? We treat each 10 ms sub-job of A as its own
job. A runs for 10 ms, COMPLETES, and then B starts running. When I/O is
complete, the new sub-job for A preempts B and so on...

Each CPU burst is treated as its own job.

### Final Assumption

Finding an optimal scheduler after relaxing the final assumption is difficult.
The OS typically knows little about each job. AKA, a scheduler knows little
about the future. To solve this, use the past to predict the future with a
*multi-level feedback queue*.
