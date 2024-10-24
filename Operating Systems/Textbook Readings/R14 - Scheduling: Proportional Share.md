## Scheduling: Proportional Share

Different type of scheduler called *proportional-share* scheduler or a
*fair-share* scheduler that, instead of optimizing for turnaround time or
response time, tries to guarantee that each job obtains a certain percentage of
CPU time. 

An early example is called *lottery scheduling*: every so often, hold a lottery
to determine which process should get to run next; processes that should run
more often have a higher chance to win. 

### Basic Concept: Tickets Represent Your Share

*Tickets* represent the share of a resource that a process should receive. The
percent of tickets that a process has represents its share of the system
resource in question. 

For example, if A has 75 tickets and B has 25 tickets, we would like for A to
receive 75% of the CPU and B 25% of the CPU.

The scheduler should know how many total tickets there are (100). It then picks
a ticket from 0 to 99. 

Advantages of randomness:  
- Avoids strange corner-case behaviors that a more traditional algorithm may
  have to handle  
- Lightweight, requires little state to track alternatives  
- Random is quite fast  

### Ticket Mechanisms

Lottery scheduling has different mechanisms to manipulate tickets. One way is
with the concept of a *ticket currency*. Currency allows a user with a set of
tickets to allocate tickets among their own jobs in whatever currency they would
like. The system then automatically converts the currency into the correct
global value. 

For example, say users A and B each have 100 tickets. A runs two jobs, A1 and
A2, each given 500 tickets (out of 1000 total) in A's currency. User B only runs
1 job and gives it 10 tickets (out of 10 total). The system converts A1's and
A2's allocation from 500 each in A to 50 each in the global currency. B1's 10
tickets is converted to 100 tickets. 

Another useful mechanism is *ticket* transfer. With transfers, a process can
temporarily hand off its tickets to another process. 

Lastly, *ticket inflation* can be useful. With inflation, a process can
temporarily raise or lower the number of tickets it owns. This can be taken
advantage of by greedy processes, so this only applies in environments where a
group of processes trust each other. 

### Implementation

The implementation is rather simple. All that is needed is a good random number
generator to pick the winning ticket. We can keep processes in a list, each
assigned a certain number of tickets. This list should be sorted from highest
number of tickets to lowest, which doesn't affect the randomization algorithm
but does make it so that less iterations are needed when iterating through the list.
If winning ticket 300 is selected, then a counter will go through each job in
the list and add their tickets. Once the counter is greater in value than the
winninng ticket (for example, adding process D makes counter go from 250 to 400,
which is greater than 300), then the iteration breaks and process D is pointed
to as the winner.

When job length isn't long, average fairness can be low. As jobs run for a
number of time slices, the lottery scheduler approaches the desired fair
outcome.

### Stride Scheduling

Randomness gets us a simpe and approximately correct scheduler, but not all the
time. The solution to this is *stride scheduling*, where each job has a stride
(the inverse in proportion to the number of tickets it has). To get the stride
of each job, divide total tickets by number of tickets assigned to job. For
example, if there are 10,000 total tickets and job A has 50 tickets, then its
stride is 200. 

Every time a process runs, a *pass value* is incremented by its stride. The
scheduler chooses the process with the lowest pass value to run.

Lottery scheduling achieves the right proportions probabilistically over time
but stride scheduling gets them exactly right.

Why use lottery scheduling instead of stride scheduling? Stride scheduling has a
global state whereas lottery scheduling doesn't. If a new process is added
during the scheduling cycle, then what pass value does it have? If 0, then it
will hog the CPU until it catches up to the rest of the jobs.

### Linux Completely Fair Scheduler (CFS)

The CFS implements fair scheduling but in a highly efficient and scalable
manner. Whereas most schedulers are based around the concept of a fixed time
slice, CFS operates differently. Each process has a *vruntime* or *virtual
runtime*. As each process runs, it accumulates vruntime. When a scheduling
decision occurs, CFS picks the process with the lowest vruntime to run next.

When to stop current process and run next one? Too often, fairness increases but
at cost of performance. Not often enough, performance increases but at cost of
fairness. 

CFS uses many metrics to determine when to stop and run processes.  
- *Sched_latency*: how long a process should run before considering a switch. A
  typical value is 48 ms. CFS divideds 48/(*n* processes) to get time slice for
  a process.  
- *Min_granularity*: minimum value that a time slice will ever be set to.
  Usually something like 6 ms.  
- *Niceness*: each process has a *nice* level that goes from -20 to +19 with a
  default of 0. Nice level determines priority or weight, where negative
  values have higher priority and positive values have lower priority. You would
  use a chart to convert nice level to weight. For example, nice of 0 is weight
  of 1024.

Time slice formula for a process =  
`(weight_of_process / sum_of_weights) * sched_latency`

With weighting, vruntime calculation is also different. `runtime_process` is the
actual run time that a process has accumulated in the current timeslice.
`default_weight` is already mentioned, 1024. vruntime of a process after a
timeslice is =   
`vruntime_process + (default_weight / weight_process) * runtime`

For example, A has a weight of 3121 (nice level of 5) and lets say its timeslice
is 10 ms, then its vruntime increases by `(1024 / 3121) * 10 = ~3.33`

Instead of lists, which don't do well when scaled, CFS stores processes in
red-black trees. It only keeps running or runnable processes in it.

If a job goes to sleep due to I/O for example, its vruntime is set to the
minimal value in the red-black tree when awoken so it doesn't hog the CPU due to
a low vruntime.
