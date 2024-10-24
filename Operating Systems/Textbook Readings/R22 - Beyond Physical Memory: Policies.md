## Beyond Physical Memory: Policies

### Cache Management

We can view main memory as a cache since it holds a subset of all virtual pages.

We want to minimize *cache misses* or number of times to fetch a page from the
disk. Alternatively maximize *cache hits*. 

*Average memory access time (AMAT)* for a program is `T_m + (P_miss * T_D)`  
- `T_m` represents the cost of accessing memory  
- `T_D`  represents the cost of accessing disk
- `P_miss` represents the probability of not finding the data in the cache (a
  miss). Hit rate is `1-P_miss`

Types of cache misses:
- Compulsory (or cold-start) miss: cache empty to begin with  
- Capacity miss: cache ran out of space and had to evict item to bring a new
  item in
- Conflict miss: limits on where an item can be placed in the hardware cache due
  to set-associativity

### Optimal Replacement Policy

Optimal policy is called MIN and leads to the fewest number of misses overall.
Replacing the page that will be accessed furthest in the future is the optimal
policy and results in fewest cache misses.

The future is not generally known, so you can't build MIN. We have to
approximate it.

### Simple Policy: FIFO

Pages that enter earlier get evicted first. They are placed on a queue. FIFO
does notably worse than optimal.

### Simple Policy: Random

Random page is selected to evict. How it does depends on luck. Sometimes, random
is almost as good as MIN and sometimes its far worse.

### Using History: LRU

Use the past as our guide. If a page was accessed recently, it is likely to be
accessed again. The *Least-Frequently-Used (LFU)* policy evicts the least
frequently used page. The *Least-Recently-Used (LRU)* policy evicts the least
recently used page. They are based in the idea of locality.


### Workload Examples

When there is no locality in the workload, FIFO, Random, and LRU all perform the
same when measuring hit rate vs cache size.

"80-20" workload: 80% of references made to 20% of pages. FIFO and Random do
decent but LRU does better but not as good as optimal when measuring hit rate vs
cache size.

Last workload example: looping sequential. Start at page 0, go to page 49 and
then go back again to 0. This one is the worst for FIFO and LRU. Random does
reasonably well but not as good as optimal. Random tends to avoid weird
corner-cases.

### Implementing Historical Algorithms

LRU does generally does better than FIFO and Random. How to implement it? 

Not easy. This is because, upon each page access, some data structure must be
updated so that the page moves to the front of the list. Since this is done on
every memory reference, this can bog down performance. 

Alternatively, the
hardware could store a time field for each page. When a page is accessed, its
current time would be set. Then the OS scans all time fields to find the least
recently used. This also doesn't work because looking through all these time
fields will take forever. 

### Approximating LRU

LRU must be approximated to limit overheads, which is what many systems do. This
requires a *use bit* or *reference bit*. There is one use bit per page and they
all live in memory somewhere (like page tables). When a page is referenced (read or written), the use bit is set by the hardware to 1. 

Many ways for the OS to use this use bit to approximate LRU. 

Simple approach: *clock algorithm*. Imagine pages arranged in a circular list,
like a clock. A *clock hand* would start by pointing to a page. If the page's
use bit is 1, then it gets reset to 0 and the clock hand moves on. The algorithm
continues until it finds a use bit set to 0. 

Some clock algorithms prefer to evict pages that haven't been modified (clean
pages rather than dirty pages). To support this behavior, the hardware should
include a *modified bit* or *dirty bit*. 

### Other VM Policies

Page replacement isn't the only policy used for VM. *Page selection* refers to
the OS deciding when to bring a page into memory.

For most pages, the OS brings the page into memory when it is accessed (called
*demand paging*). The OS can also guess a page is going to be used and bring it
in ahead of time (called *prefetching*). 

Another policy determines how the OS writes pages out to disk. They could be
written out one at a time but many systems collect a number of pending writes
together and write them to the disk in efficient write. This is called
*clustering*.

### Thrashing

When demands of running processes exceeds available physical memory, the system
will be constantly paging (called *thrashing*). 

Early systems tried to address this by not running a subset of processes. The
reduced set of *working sets* (pages that are actively being used) fit in memory
and thus can make progress. This is called *admission control*. 

Some versions of Linux and other systems run an *out-of-memory killer* when
memory is oversubscribed. This daemon chooses a memory-intensive process and
kills it.
