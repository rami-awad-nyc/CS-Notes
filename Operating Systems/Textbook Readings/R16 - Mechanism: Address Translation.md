## Mechanism: Address Translation

In CPU virtualization, LDE (*limited direct execution*) was used: let the
program run directly on the hardware and the OS only interferes at certain key
points (like a system call or timer interrupt) to maintain control and
efficiency. In memory virtualization, a similar strategy is used. 

The technique used is referred to *hardware-based address translation* or just
address translation. The hardware transforms each memory access (fetch, load,
store), changing the virtual address provided to a physical one where the data
is actually stored. 

### Assumptions:

For first attempts, we assume user's address space must be placed contiguously
in physical memory, the size of address space is less than physical memory, and
each address space is the same size. Of course, these are entirely unrealistic
but they will be relaxed as we go.

### Dynamic (Hardware-based) Relocation

*Dynamic relocation* or *base and bounds* is a simple idea introduced in the
1950s: one hardware register in CPU called *base* and another called *bounds* or
*limit*. 

When a program starts running, the OS decides where in physical memory it should
be loaded and sets the base register to that value. Now when any memory
reference is generated by the CPU, it is translated with `physical memory =
virtual address + base`. This is so virtual addresses can start at 0. 

The bounds register helps with protection. The processor will check that the
memory reference is within bounds to make sure it is legal. The value stored
here will be the size of the address space.

This is a simple method of address translation. Generally, people call the part
of the CPU that helps with address translation the *memory management unit*
(MMU). 

### Hardware Support: A Summary

Two different CPU modes are needed:  
1. The OS runs in *privileged* or *kernel* mode, where it has access to the
entire machine  
2. Applications run in *user* mode where they are limited in what they can do.

A single bit indicates which mode the CPU is in. This is stored in some kind of
*processor status word*. The mode switches in special occasions, like a system
call or exception or interrupt.

Each CPU must also have a base register and a bound register in the MMU. The
hardware should provide privileged (only in kernel mode) instructions to modify
these registers, allowing the OS to modify them. 

Lastly, the CPU has to generate exceptions when a user program tries to access
memory illegally (out of bounds). An out of bounds handler should run.

### OS Issues

Where does the OS take action here?

Since address spaces are the same size and smaller than physical memory, the OS
has to search a data structure (called a *free list*) to find room for a new
address space when a process is created.

The OS also has to do work when a process is terminated (reclaiming its memory).

The OS also has to perform a few steps when a context switch occurs. The OS must
save and store the values in the base and bounds registers when it switches
between processes, typically in the PCB. 

OS must also provide exception handlers or functions to be called. At boot time,
these handlers are installed (via privileged instructions). 

### Summary

Base and bounds is a simple way to virtualize memory with transparency,
efficiency, and protection. However, if a process' stack and heap are not used,
then physical memory is wasted (called *internal fragmentation*). The solution
is *segmentation*. 
