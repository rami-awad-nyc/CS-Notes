## The Process

A *process* is an instance of a running program. Programs on their own are just
instructions and static data. The OS takes these bytes and makes programs
useful.

### CPU Virtualization

Time sharing: time with a resource split amongst many entities (CPU)  
Space sharing: resource divided in space amongst many entities (memory)  

To make it seem as if there are many CPUs, the OS allows time sharing of the
CPU:  
`Process 1: running ------- running`  
`Process 2: ------- running -------` 

It seems as if both processes are running at the same time, but in reality, they
each get a time slice with the CPU. The more processes running, the slower they
run

*Mechanisms* are low-level protocals or methods that implement a functionality
- Example: context switch mechanism gives the OS the ability to stop running a
  process and start running another on the CPU

*Policies* are algorithms for making somme kind of decision within the OS
- Given a number of processes to run on the CPU, how does the OS choose which to
  run? A scheduling policy

### Process API

A process' machine state is what it can read or update when it's running
- The memory that the process can address (called its address space)
- Registers - including special ones like the stack pointer (RSP), frame pointer
  (RBP), and program counter (RIP).
- Persistent storage devices

Some process APIs that should be present on any modern OS:
- *Create*: OS should have a method to create new processes
- *Destroy*: OS should be able to provide an interface to kill processes
- *Wait*: OS should have a method to wait for a process to stop running
- *Misc Process Control*: Other methods for control (like pause for some time)
- *Status*: OS should have a method to get status info about a process

### Programs to Processes

Program on disk exists in some kind of executable format:
- Program on disk contains instructions and static data in machine code

Program to Process Steps:
1. *Load* the program's code and static data into memory into the process' address
   space.
2. Allocating memory for the *run-time stack* (or just stack) and initialize
   stack with the `main()` function's parameters (argc and argv).
3. Allocating memory for the heap, which starts out small but will grow using
   malloc().
4. Some I/O setup --- for example, in UNIX systems, processes start with three
   open file descriptors: standard input, standard output, standard error
5. Last step: jump to main() routine and transfer control of CPU to process.

After these steps, the process begins its execution.

Two ways of performing the loading process:
- Eagerly: loading code and static data all at once before program executes
- Lazily: loading pieces of code and data as they are needed during execution  
Modern OSs perform loading lazily.

### Process States

A process can be in one of three states:
- Running: process is running on a processor and executing instructions
- Ready: process is ready to run but OS chose not to run it at the moment
- Blocked: process performed an operation that makes it not ready to run until
  some event takes place

Process being moved from ready to running means the process has been
*scheduled*. Process being moved from running to ready means the process has
been *descheduled*.

Only one process can be in the running state at a time for each core of the CPU!

### Data Structures

Every OS has a *process list* or task list that tracks the stack of each
process. The info for each process is stored in a *Process Control Block* (PCB)
or process desriptor. The process list is really a collection of PCBs.
