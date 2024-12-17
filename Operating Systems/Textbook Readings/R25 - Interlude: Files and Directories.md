## Interlude: Files and Directories

A persistent storage device such as a hard drive or SSD stores information for a
very long time. 

### Files and Directories

A *file* is a linear array of bytes, each of which you can read or write. The
low-level name of a file is the *inode number*. 

A *directory* contains a list of (user-readable name, low-level name) pairs.
Directories also have inode numbers. 

By placing directories inside of directories, we can build directory
hierarchies. Hierarchy starts at *root directory* (`/` in Unix). 

### Creating Files

We can create files with `open` system call and passing in `O_CREAT` flag.  
`int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);`
- `O_CREAT`: creates the file if it doesn't exist
- `O_WRONGLY`: can only be written to 
- `O_TRUNC`: if it exists, reduce size to 0 bytes, removing all content

`open()` returns a *file descriptor*, private integer per process. Some
strucutre kept in the `proc` structure to store all file descriptions for each
process.

### Reading and Writing Files

`strace` traces every system call made by a program while it runs. 
A file is opened for reading, not writing, if the `O_RDONLY` flag is used.

Each process, by default, has 3 files open: standard input (which the process
can read to receive input), standard output (which the process can write to in
order to dump info on the screen), and standard error (which the process can
write error messages to). FDs are 0,1,2 respectively.

`read()` and `write()` system calls used to read and write given an fd. For
writing to the console, `printf()` is used to handle all formatting, which
itself calls `write()`. When a file is no longer needed, use `close(fd)` to
close it.

### Reading and Writing, But Not Sequentially

Sequential access: reading a / writing to a file from beginning to end.

Use `lseek()` to read/write to a specific offset within a file:  
`off_t lseek(int fd, off_t offset, int whence)`
- whence: if `SEEK_SET`, the offset is set to offset bytes. If `SEEK_CUR`, the
  offset is set to its current location plus offset bytes. If `SEEK_END`, the
  offset is set to the size of the file plus offset bytes.

The OS tracks a current offset which tracks where a read or write will begin
reading from. This offset is stored in `struct file` and is added to when a read
or write of N bytes takes place (N is added to the current offset) or when
`lseek` is called, which changes the offset.

`read()` returns 0 if the current offset surpasses the size of the file. Also,
when a file is first opened, its current offset is set to 0.

### Shared File Table Entries: fork() and dup()

If a process reads the same file at the same time as another process, then each
will have its own entry in the open file table. Current offset will not be
shared.

However, if a parent process creates a child process with `fork()`, then the
current offset is shared. In other words, when a file table entry is shared, its
reference count is incremented. When both processes close the file, the entry
will be removed.

The `dup()` call allows a process to create a new fd that refers to the same
open file as an existing descriptor. 

### Writing Immediately With fsync()

The file system buffers writes to the disk in memory for some time (say 5
seconds). `fsync()` for a particular fd will force all *dirty* (not yet written)
data to the disk. 

### Memory Mapping

Memory mapping is an alternative way to access persistent data in files. The
`mmap()` call creates a correspondence between byte offsets in a file and
virtual addresses in the calling process. The former is called the *backing
file* and the latter its *in-memory image*. The process can then access the
backing file using CPU instructions to the in-memory image.

### Renaming Files

We can rename files with `mv` which uses the `rename()` system call. `rename()`
is atomic with respect to crashes: eitehr a file is renamed or it isn't.

Editors would write out a new version of a file being changed under the
temporary name (foo.txt.tmp), force it to the disk with `fsync()`, and then
rename the file to the original name. This ensures that the file update is
atomic.

### Getting Info About Files

*Metadata* is info about a file. To see the metadata for a certain file, use the
`stat()` or `fstat()` system calls. They take a pathname or fd to a file and
fill in the stat structure, which has a lot of info like inode number, UID,
GID, time of last access, etc.

Each file system keeps this type of info in a structure called an *inode*.

### Making and Reading Directories

To create a directory, use `mkdir()` system call.

`ls` uses three calls to print the content of a directory: `opendir()`,
`readdir()`, and `closedir()`. Each directory has directory entries called
dirents that store info about each entry in the directory, most stuff like
filename and inode number. 

To delete a directory, you call `rmdir()`, which has the requirement that the
directory is empty before deletion.

### Hard Links

The `link()` system call takes two arguments, an old pathname and a new one.
When you link a new file name to an old one, you essentially create another way
to refer to the same file.

`link()` creates another name in the directory and refers to the same inode
number of the original file. The file itself is not copied, but you now have two
human-readable names that refer to the same file.

To remove a file from the file system, call `unlink()`. When the file sytem
unlinks a file, it checks the reference count of the inode number. If the
reference count decreases to zero, the file system frees the inode.

### Symbolic Links

Symbolic or soft links are similar to hard links but are actual files
themselves. Whereas hard links are filenames pointing to the inode of the
original file, soft links are files that store the pathname to the original
file as its data.

We have a *dangling reference* if we remove the original file, which causes the
link to point to a pathname that no longer exists.

### Permission Bits & Access Control Lists

UNIX *permission bits* can be seen why typing `ls -l` for a file.

The leftmost block of text represents the permission bits. There are 3
groupings: what the owner of the file can do, what someone in a group can do to
the file, and finally, what anyone (other) can do. The abilities for all 3
include the ability to read the file, write it, or execute it.

Use `chmod` to change permission bits for files.

For directories, the execute bit behaves differently to files. It enables
someone to do things like change directories into the given directory, and, in
combination with the writable bit, create files therein.

### Making and Mounting a File System

To make a file system, most file systems provide a tool, usually referred to as
`mkfs` that performs this task. Give `mkfs` a device and a file system type
(like ext3) and mkfs will write an empty file system, starting with a root
directory, onto the device.

The `mount` program (which calls `mount()`) takes an existing directory as a
target mount point and pastes a new file system onto the directory hierarchy at
that point.
