## Beyond Physical Memory: Mechanisms

So far we have assumed that an address space is unrealistically small and fits
into physical memory. Thus far, we assumed all pages reside in physical memory.
The OS needs a place to stash away portions of address spaces that aren't
currently in demand: usually a hard disk drive (but can be something similar
like an SSD).

### Swap Space

We first must reserve space on disk for moving pages back and forth: called
*swap space*. The OS will need to remember the *disk address* of a given page.

Swap space is generally *very* large and measured in pages. If the number of
valid virtual pages is greater than the number of physical pages, then the
remainder of the virtual pages are stored in the swap space. 

### The Present Bit

When the hardware looks into a PTE, it may find that the page isn't present in
physical memory despite being valid. Whether it is present or not is determined
by the *present bit*.

If present bit is 0, then a *page fault* occurs. When this occurs, the OS
invokes a *page-fault handler*. 

### The Page Fault

The OS needs to swap the page into memory from the disk. Where? When present bit
is 0, the bits for the PFN are instead used to represent the disk address. So
when a page fault occurs, the OS looks at the PTE of the page.

After the disk I/O completes, the OS updates the page table with the PFN of the
page in memory and the present bit set to 1. 

While the I/O is in flight, the process will be in the *blocked* state.

### What If Memory Is Full?

What if we need to remove a page from the memory to bring in a page from the
disk? So instead of a page in, we now have a swap.

First, the OS pages out one or more pages to make room for the new page. This
policy is called the *page-replacement policy*. 

### When Replacements Really Occur

In reality, the OS doesn't wait until memory is full to evict pages. It instead
has a small amount of free memory. Most OSs have a *high watermark (HW)* and
*low watermark (LW)* to help decide when to start evicting pages from memory.

When the OS notices fewer than LW pages available, a background thread
responsible for freeing memory runs. The thread evicts pages until HW pages are
available. The thread is called the *swap daemon*.

Many systems will clutter or group many pages and swap them all at once to
increase efficiency. 
