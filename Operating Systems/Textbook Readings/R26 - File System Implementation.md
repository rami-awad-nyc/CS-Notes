## File System Implementation

For these notes, we use a simple file system implementation. It is pure
software. Unlike CPU and memory virtualization, no hardware is needed.

### Overall Organization

Disk is divided into *blocks*. Common size is 4KB. The blocks are addressed from
0 to `N - 1` in a partition size of `N` 4KB blocks.

Most of the space in a file system is user data. This region is called the *data
region*. 

The filesystem has to track info about each file. To store this info, file
systems usually have a structure called an *inode*.

We have a portion of the disk dedicated to inodes through an *inode table*,
which simply holds an array of on-disk inodes. Inodes are not that big. If there
are 256 bytes per inode, then a 4KB block can store 16 inodes. The number of
blocks for the inode table times the number of inodes per block represents the
max number of files in a file system.

There needs to be a way to track whether inodes or data blocks are free or
allocatd. We could use a *free list* that points to the first free block, which
then points to the next free block, etc. A popular structure is using a
*bitmap*, one for the data region and one for the inode table. Each bit in a
bitmap indicates whether a block is free (0) or in use (1). 

The first block in a disk is called the *superblock*, which contains infolike
how many inodes and data blocks are in the system, where the inode table begins,
etc.

### File Organization: The Inode

Each inode is referred to by a number, called the inode number. 

In an example, let's say the system wants to read inode number 32. It would add
`32*4KB` to the `inodeStartAddr` which is found in the superblock. Let's say
this is 12KB. So the inode is located at address 20KB. Since sectors are 512
bytes, the sector of the inode block is (20KB)/512 = 40.

All the info inside an inode is called *metadata*. 

How does the inode refer to where data blocks are? One approach is to store disk
addresses or *direct pointers* inside the inode; each address points to one disk
block that belongs to the file. 

Alternatively, but not as popular, are *extents*, which is simply a disk pointer
plus a length. However, this requires that files are stored contiguously, which
isn't always the case. On the other hand, using extents reduces the size of
metadata.

Another alternative is to use a linked list. Instead of having multiple pointers
inside an inode, just one is needed to point to the first block of the file. To
handle bigger files, add a pointer to the end of the first block of the file and
so on. However, this makes random access difficult and clearly has performance
issues for big files. Some systems keep an in-memory table of link info, where
each entry is the address of the next block in the file. This is the basic
structure of the *file allocation table*.

### Multi-Level Index

To support bigger files, a special pointer known as an indirect pointer is used.
Instead of pointing to a block that contains user data, it points to a block
that contains more pointers, each of which points to user data. An inode may
have some fixed number of direct points and a single indirect pointer. If the
file grows large enough, an indirect block is allocated and the inode's slot for
an indirect pointer is set to point to it.

To support even larger files, add another pointer to the inode: the double
indirect pointer, which points to a block that contains indirect pointers.

This supports files up to 4GB in size `(12 + 1024 + 1024^2) x 4KB`.

We can even have a triple indirect pointer, etc. This imbalanced tree is called
a multi-level index. Why use this system? Because most files are small.

### Directory Organization

Directories are treated as a special type of file. A directory has an inode but
with a type of 'directory' rather than 'file'. 

### Free Space Management

When we create a file, the file system searches through the bitmap for an inode
that is free and allocates it to the file. The file system marks the inode as
used (1) and updates the on-disk bitmap with the correct info. A similar set of
activites takes place when a data block is allocated.

Some systems like `ext3` look for a sequence of blocks to guarantee some
contiguity. This is a *pre-allocation* policy.

### Reading and Writing Access Paths

When you call `open()` for reading,  the file system first traversesthe pathname
to locate the desired inode. It starts at the root, which has its own inode
number that is well-known by the file system (usually 2). After reading in the
inode number of 2, the system uses the pointers inside to find the root
directory's data blocks. Inside of these data blocks are the directory entries
for the root. A recursive traversal of the pathname proceeds and the file's
inode is read into memory. The system does a permissions check, allocates an fd
for the file in the open-file table, and returns it to the user.

When you issue `write()` calls to the update a file, 5 I/Os are generated: one
to read the data bimap, one to write the bitmap (update for the newly allocated
block), two more to read and write the inode (updated with location of new
block), and finally one to write the actual block.

### Caching and Buffering

Reading and writing files can be expensive and incurring many I/Os to the disk.
As such, most systems use DRAM to cache important blocks.

Early systems used a *fixed-size cache*, but this can be wasteful. Modern
systems use a *dynamic partitioning* approach. Many OS's integrate virtual
memory and file system pages into a *unified page cache*. 

With a cache, the first open for a read may generate a lot of I/O, but
subsequent opens will mostly hit the cache and thus no I/O is needed.

Caching isn't as effective for writes because write traffic has to go on the
disk regardless. However, *write buffering* has many benefits: by delaying
writes, the system can batch updates with less total I/Os. Most systems buffer
writes in memory for anywhere between five and thirty seconds. Of course, if a
crash occurs, this data may be lost. Tradeoff between durability and
performance. 
