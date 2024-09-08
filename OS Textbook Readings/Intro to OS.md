## INTRO TO OS

Von Neumann model of computing
- For each instruction, the processor: fetches instruction from memory → decodes
  instruction (figures out what the instruction is) → executes instruction →
  moves on to next instruction

The OS is a program that acts as an intermediary between the system (or computer)’s hardware and software; here are its functions:
- The OS makes the system operate efficiently through visualization — taking
  physical resource such as processor, memory, disk, and transform into abstract
  and easier-to-use virtual form of itself
	- OS often called a virtual machine
- OS provides APIs or interfaces or system calls to applications → OS provides a
  standard library to applications
- OS is a resource manager → efficiently manages resources amongst processes

Virtualizing the CPU: the illusion of turning a single CPU into a seemingly infinite number of CPUs
Virtualizing memory: each process has its own private virtual address space (can
simply be called its address space) ← appears as if each process has all the
physical memory to itself, although physical memory must be shared amongst the processes

Concurrency represents numerous things running at the same time:
- The OS juggling many processes running at the same time
- Programs juggling many threads running at the same time
	- Threads are functions running at the same time (they all share the
	  same address space)

Persistence: the theme that data needs to be stored for long periods of time
- Hard drives and SSDs used to store long-term data
- Which software in the OS manages the disk? The file system
	- Unlike CPU and memory abstractions, file system is not isolated for
	  each process because files tend to share info
	- Programs make system calls into the OS to interact with other files
	  (the OS handles system calls in the file system)
		- System calls provide a simple way to access devices


