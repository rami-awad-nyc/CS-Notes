## Crash Consistency: FSCK and Journaling

How to update persistent data structures in the presence of a *power loss* or
*system crash*?

Approach taken by older file systems: *fsck* or the *file system checker*.
Another approach: *journaling* or *write-ahead logging*. 

### Detailed Example

Assume a simple workload: append a single data block to an existing file,
calling `lseek()` to move the file offset to the end of the file and then
issuing a single 4KB write to the file before closing it. Also assume we use an
inode bitmap with just 8 bits, a data bitmap with just 8 bits, and 8 total
inodes across 4 blocks and 8 total data blocks.

To append a block, the file system must perform 3 separate writes to the disk:
one each for theinode, bitmap, and data block. These writes are not immediate.
The dirty data sits in memory for some time and then gets written to the disk.
What happens if there is a crash in this time?

### Solution 1: File System Checker

Early systems decided to let inconsistencies happen and then fix them after
rebooting. One of the tools that does this is `fsck`. However, if the metadata
is consistent but the write to the file block fails, then there is no problem
here from the perspective of `fsck`. The only goal is to make sure file system
metadata is consistent.

`fsck` runs before the file system is mounted:
1. First checks if the superblock looks reasonable. May use alternative if
corruption found (basic stuff like making sure the file system size is greater
than number of blocks allocated).
2. Scans the inodes, indirect blocks, double indirect blocks, etc. Uses this
knowledge to produce a correct version of the allocation bitmaps. 
- Inode state, link count, duplicate pointers, bad block pointers (points to
  something outside its valid range), directory checks

Performance of `fsck` is very slow... may take hours.

### Solution 2: Journaling (or Write-Ahead Logging)

When updating the disk, before over-writing the structures in place, first write
down a little note in a well known location on the disk describing what you are
about to do. If a crash occurs, go back to the note and try again.

A journal is a data structure that occupies some space on a file system (think
of it as being before group 0). 

Back to the example from before (with 3 writes). There are 5 writes to the log.
The first is `TxB` or "transaction begin", which tells us about this update
(i.e. final addresses of the 3 blocks) and includes a transaction identifier
(TID). The middle 3 blocks contain the exact contents of the blocks themselves.
This is called *physical logging*. The final block is `TxE` or "transaction
end".

Once this transaction is safely on the disk, we can actually write to the disk,
called *checkpointing*. So:
1. Journal write
2. Checkpoint

What about crashes that occur during journaling? The scheduler might write `TxE`
to the journal before writing a data block write to the journal. If power is
lost before the data block is written, then the journal would contain what
appears to be a normal transaction that ends with `TxE`. 

To avoid this problem, the file system writes all blocks except the `TxE` block
to the journal, issuing these writes all at once. Once the writes are complete,
then `TxE` is written to the journal. `TxE` should also be 512 bytes so that it
is atomic (to match sector size).
1. Journal write
2. Journal commit (`TxE`... transaction is said to be committed)
3. Checkpoint

How to use the journal to recover from a crash? 
- If a crash happens before the commit, then the update is skipped entirely. 
- If the crash happens after the transaction has committed to the log, but
  before the checkpoint is complete, then the file system can recover the update
  as follows: when the system boots, the file system recovery process will scan
  the log and look for transactions that have committed to the disk; these
  transactions replayed in order, with the file system attempting to write out
  the blocks in the transaction to their final locations. This is called *redo
  logging*.

To prevent writing the same blocks over and over (for example, creating files in
the same directory -- located on the same inode block), some file systems do not
commit each update to disk one at a time. They instead buffer all updates into a
global transaction.

### Making the Log Finite

The log is treated as a circular data structure, reusing it over and over. Once
a transaction has been checkpointed, the file system should free the space it
was occupying.

A journal superblock contains enough info to know which transaction have not yet
been checkpointed. 

Final steps:
1. Journal write
2. Journal commit
3. Checkpoint
4. Free transaction

### Metadata Jouranling

For each write to the disk, we now also write to the journal, which doubles
write traffic. The method used throughout this section is *data journaling*, but
a simpler form is called *ordered journaling* (or just *metadata journaling*).
It is nearly the same except that user data isn't written to the journal.

To prevent garbage data from being used, some systems write data blocks of
regular files to the disk first before metadata is written to the disk.
1. Data write
2. Journal metadata write
3. Journal commit
4. Checkpoint metadata
5. Free transaction

This is used more than data journaling.

### Solution 3: Other Approaches

*Copy-on-write* (COW) is used in a number of popular file systems. COW places
new updates to previously unused locations on disk rather than overwriting files
or directories in place. After a number of updates are complete, COW systems
flip theroot structure of the file system to include pointers to the newly
updated structures.
