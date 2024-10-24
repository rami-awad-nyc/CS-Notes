## Lock-based Concurrent Data Structures

How to add locks to a data structure so that many threads are able to access the
structure concurrently?

Generally speaking, you want a lock to be one of the fields for a data
structure. You then lock at the beginning of each functions (or method for C++)
and unlock at the end of each function. After this coarse-grain strategy is
implemented, you can try to optimize the program by adding more locks.

A *scalable* data structure has threads completing more work on multiple
processors without losing performance. 

### Counter

A counter has a field called `count`. It can increment this value, decrement it,
and obtain the value. This data structure is not scalable if multiple processors
are present. The solution is (called an *approximate counter*) to have one local
lock and local count per processor as well as one global lock and global count.
As threads run on the separate processors, the local counts are incremented.
Every once in a while, all the local counts are added to the global count and
reset to 0. 
- How often to add to global count? Based on a threshold called `S`. If `S` is
  smaller, global count is more accurate but the data structure is less
  scalable. If `S` is bigger, global count is less accurate but data structure
  is more scalable.

### Linked Lists

When inserting in a linked list, bugs can occur if `malloc` fails and there are
locks surrounding it. Assuming `malloc` is thread-safe, we can fix this by
locking after `malloc` and some local variables and unlocking at the end of the
function. This is also valid because variables declared and initialized inside a
function aren't shareable across threads unless passed as parameters into functions.

Scaling linked lists: use a technique called *hand-over-hand locking* or *lock
coupling* to scale with more processors. Basically, you have one lock per node
of the list. When accessing the next node, you grab the next node's lock and
then release the current node's lock. In practice, this technique has lots of
overheads that hurt performance.

### Queues

Michael and Scott created a concurrent queue that has two locks: one for the
head and one for the tail. Basically, use the tail lock for the `enqueue`
function and the use the head lock for the `dequeue` function. This type of
queue works but can be developed more with condition variables (next chapter).

### Hash Tables

A concurrent hash table is straightforward and is built using the linked lists
mentioned previously in these notes. There is one lock per hash bucket (each of
which is represented by a list). 
