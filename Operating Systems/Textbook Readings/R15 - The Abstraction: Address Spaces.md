## The Abstraction: Address Spaces

### Multiprogramming and Time Sharing

Multiprogramming is when many programs run at a given time. As interactivity and
the need for multiprogramming increased, *time sharing* was born. One way was to
run one process for a bit, giving it access to full memory, then stop it and
save ALL of its state to a disk (including all of its physical memory!) and load
another process' state. This was slow and inefficient, especially as memory got
bigger. 

To make time sharing more efficient, we want to leave processes in
memory while switching between them. However, letting many programs sit in
memory makes *protection* an issue

### The Address Space

The abstraction of physical memory is called the *address space*, and it is a
process' view of memory in the system. This includes the stack, heap, and code.
The program code starts at 0 and doesn't change. The stack and the heap grow and
shrink while a program runs. The address space can be arranged in different ways
(like if there are multiple threads in a process). 

The process isn't actually in physical memory from addresses 0 to ...   
This is an *abstraction* that the OS provides to the running program. 

When the OS does this, we say the OS is *virtualizing memory* because the
running process thinks it is loaded into memory at memory address 0 and has a
very large address space, when in reality, it is different.

When you print out addresses in C, you are actually printing virtual addresses.

How can the OS build this abstraction of a private, potentially large address
space for multiple running processes (all sharing memory) on a top of a single
physical memory? 

### Goals

*Transparency*: the OS should implement virtual memory in a way that is
invisible to the running program. The program shouldn't be aware of the fact
memory is virtualized. 

*Efficiency*: Virtualization shouldn't make programs run much more slowly and
shouldn't take up too much actual memory for structures needed to support
virtualization. 

*Protection*: Processes should be protected from one another as well as the OS
from processes. 
