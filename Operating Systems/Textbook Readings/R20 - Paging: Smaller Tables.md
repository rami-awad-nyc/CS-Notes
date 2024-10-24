## Paging: Smaller Tables

Linear page tables get too big. 

### Simple Solution: Bigger Pages

By increasing the size of pages, we need less page tables entries to cover all
pages. However, big pages lead to waste within pages (*internal fragmentation*).
Applications allocate pages but only use little bits and pieces of each. Thus,
most systems use small page sizes (4KB is most common).

### Multi-level Page Tables

A multi-level page table turns a linear page table into something like a tree.
Many modern systems employ it.

Chop up page table into page-sized units. If an entire page of PTEs is invalid,
don't allocate that page of the page table at all. To track whether a page of
the page table is valid, use a new structure called a *page directory*. It tells
you where a page of the page table is or if the page contains no valid PTEs.

In a simple two-level table, the page directory contains one entry per page of
the page table. It consists of a number of page directory entries (PDE). A
minimal PDE has a valid bit and a *page frame number* (PFN). If valid bit is 0,
that means none of the PTEs on the page represented by the PDE are valid.
Accordingly, that PDE is undefined. If valid bit is 1, then the PDE contains
PFN that point to second-level page tables.

The page directory is located at a specific physical address in memory stored in
page directory base register (PDBR).

Advantages?   
1. Only allocates page-table space in proportion to amount of address space
being used.  
2. Easier to manage memory. In a linear page table, the PTEs must reside
contiguously in physical memory. In a multi-level page table, page-table pages
can lie anywhere in physical memory.

Costs?  
1. Two loads required to get right translation (one for page directory and one
for PTE itself) rather than just one.  
2. More complex

### Example

Virtual pages 0 and 1 are for the code, 4 and 5 for the heap, 254 and 255 for
the stack. The rest are unused.
Address space of size 16K.
Page size of 64B.
PTEs are 4B.

How many bits in virtual address space?
16K is 2^14 so 14 bits.

How many bits for the offset?
64B is 2^6 so 6 bits.

How many bits for the VPN?
16K / 64B = 2^14 / 2^6 = 2^8 so 8 bits

2^8 is also 256, so there are 256 PTEs in the page table. Each PTE takes up 4B,
so the page table is 1K.

How many pages represent the page table? 1K / 64B = 2^10 / 2^6 = 2^4 = 16.
16 pages represent all 256 entries, with each page having 16 PTEs.

The page directory needs one PDE per page so the page directory has 16 PDEs.
16 = 2^4 so 4 bits are needed to represent the PDE.
The VPN is broken into 4 bits for the PDE index and 4 bits for the page table
index to determine which of the 16 PTEs within each page to look at. 

### More Than Two Levels

What if page directory gets too big?

To determine how many levels needed in a multi-level table, we determine how
many PTEs fit within a page. We then take the page directory size and divide it
by the PTE/page value. We keep doing this to the level PD until the number of PDEs
fits into one page. 

### x86-64 SUPER IMPORTANT, READ THIS FOR COMPLETE PICTURE

In x86-64, there are 4 levels of page tables:  
- L1: page directory with PDEs pointing to L2 pages
- L2: page directory with PDEs pointing to L3 pages
- L3: page directory with PDEs pointing to L4 pages
- L4: page table pointing to PPNs (L4 makes up the most number of pages)

In x86-64, virtual addresses are 64 bits but only 48 are usable (canonical
form). Page sizes are 4KB so VPNs are 36 bits. These 36 bits are divided into 9
bits each for the 4 levels. The first 9 bits are for L1 and etc... Physical
addresses are 52 bits. 

To find a PA given VA: first, take the L1 base address found in the PDBR. This is
40 bits long. Then take the 9 L1 bits from the VPN and place them after the 40
bits. The last 3 bits show that each PDE takes up 8 bytes. 

Fetch the address from above, get the PFN and set it as the first 40 bits. Then,
take the L2 bits from VPN and place them after the 40 bits... this process keeps
repeating until L4 is reached.

When L4 is reached, take the PFN and make it the first 40 bits. Then make the
original offset from the VA the next 12 bits. Now you have the physical address.

*DONE*

If all PTEs are invalid in an L4 table, then the L4 PDE is invalid in an L3 table.
If all the L4 PDEs are invalid in an L3 table, then the L3 PDE is invalid in an
L2 table.... and so on. 

If an L4 PTE changes from invalid to valid, the OS must ensure that all
higher-level tables reflect this change. It does so by allocating new physical
memory for missing page tables, updating the entries at each level to point to
these new tables, and setting the valid bits appropriately.


