## Locality and the Fast File System

The original file system for UNIX was very simple: just a superblocks, then the
inode table, then the data region.

However, the performance was terrible because the file system treated the disk
like it was random-access memory. Data being spread all over the place meant
that seek times for the disk were high. Also, free space was not properly
managed --> fragmentation.

Another problem: block sizes were too small (512 bytes). Smaller blocks minimize
internal fragmentation (waste within each block) but are bad for transfers
because each block might require a positioning overhead to reach it.

### Fast File System (FFS)

A group at Berkely made FFS to make the file system structures "disk aware". 

The first step was to change the on-disk structures. FFS divides the disk into a
number of cylinder groups. A single cylinder is a set of tracks on different
surfaces of a hard drive that are the same distance from the center of the
drive.

Imagine 10 surfaces and 5 tracks per surface. There are 5 cylinders, one for
each track per surface. The tracks in a cylinder are directly above and below
each other. 

FFS aggregates *N* consecutive cylinders into a group. Modern drives don't
really give file systems this much information, however, so systems organize the
drive into *block groups*, each of which is just a consecutive portion of the
disk's address space.

By placing two files within the same group (block or cylinder, doesn't matter),
FFS ensures that accessing one after the other will not have long seeks
across the disk. 

FFS needs to have the ability to place files and directories into a group and
track all necessary info about them inside of the group. FFS includes all
structures you might expect a file system to have within each group (space for
inodes, data blocks, bitmaps). FFS keeps a copy of the super block in each group
in case one copy becomes corrupt.

### Policies: How to Allocate Files and Directories

FFS now has to decide how to place files, directories, and metadata on the disk
to improve performance. Basically: keep related stuff together.

For directories: find the cylinder group with a low number of allocated
directories (to balance directories across groups) and a high number of free
inodes (to allocate a bunch of files). Put the directory and inode in that
group. 

For files: FFS first makes sure to allocate the data blocks of a file in the
same group as its inode. Second, it places all files that are in the same
directory in the cylinder group of the directory they are in.

### The Large File Exception

In FFS, there is an important exception to the general policy of file placement:
large files. 

For large files, FFS does the following: after some number of blocks are
allocated into the first block group, FFS places the next large chunk of the
file (those pointed to by the first indirect block) in another block group. Then
the next chunk is in a different group and so on.

Rather than stuffing the large file into one group (even if it fits), the file
is evenly spread across many groups. This hurts performance, but choosing a
chunk size carefully mitigates this. FFS places the first twelves direct blocks
in the same group as the inode. Each subsequent inode block and all blocks it
points to is placed in a different group. 

### A Few Other Notes

FFS designers were worried about internal fragmentation, so they introduced
*sub-blocks*, which were 512-byte little blocks that the file system could
allocate to small files. As a file grows, the file system continues allocating
512-byte blocks to it until it acquires a full 4KB of data. At that point, the
data is copied into a full 4KB block and the sub-blocks are freed.

However, this is inefficient, so FFS mostly uses write buffering to avoid
internal fragmentation.

Another issue: in sequential reads, after reading block 0, the FFS issues a read
to block, but it is too late: block 1 had rotated under the head and now a full
rotation is needed.

FFS solution? Skip over every other block, called *parameterization*. However,
once again, this is inefficient. Modern disks use an in-house *track buffer*, so
no need for parameterization.

Additionally, FFS introduced symbolic links and the atomic `rename()` operation.


