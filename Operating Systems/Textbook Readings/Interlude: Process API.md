## Process API

Each process has its own *process identifier* (PID). 

### Creating and Controlling Processes

`fork()` system call used to create a new process  
- The process created is almost a copy of the calling process
	- Has its own address space, registers, program counter
- The new process is called the *child* and the creator is called the *parent*

A child's PID is one more than its parent's PID.     
`fork()` itself returns 0 if the current process is the child and the child's
PID if the current process is the parent.

A CPU scheduler determines which process runs at a given moment in time.  

The parent calls system call `wait()` to delay execution until child
finishes running.  

`exec()` replaces current process with another program.  
1. `exec()` loads code and static data from the specified program and overwrites
  current code and static data.  
2. The heap and stack are re-initialized  
3. OS runs program with any arguments as `argv`

### The Shell

The shell is just a user program.    
After typing in a program/command and arguments into a prompt, the shell calls
`fork()` to create a new process and then calls `exec()` to run the program
typed into the prompt. It then calls `wait()` to wait for the process to finish.
- `bash, tcsh, zsh` are just a few examples of famous shells

When there's an output, the OS starts looking for free file descriptors. Usually
this is standard output.
- When the shell redirects output to a file, before calling `exec()`, it closes
  standard output and opens the file. Therefore, the first free file descriptor
  is the file. 
