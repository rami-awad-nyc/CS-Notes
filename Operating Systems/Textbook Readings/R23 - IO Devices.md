## I/O Devices

### System Architecture

Traditional System Architecture: CPU is connected to main memory by a *memory
bus*. Connecting to the memory  bus is the *general I/O bus*, which is the *PCI*
on modern systems. This is where high performance I/O devices are found, namely
*graphics*. Connecting to the general I/O bus are the *peripheral buses*. These
include *USB, SATA, SCSI*. Each of these peripheral buses connects a slower
device (like mouse, keyboard, USB, SSD, etc). 

Modern System Architecture: CPU is connected to memory through a *memory bus*,
the graphics card through *PCIe graphics bus*, and the *I/O Chip* through the *DMI
(Direct Media Interface) bus*. Connecting to the I/O Chip are the rest of the I/O
devices mentioned before. One or more hard drives are connected with *eSATA
buses*. Keyboards and mice are connected through *USBs (Universal Serial Bus)*,
and high performance I/O devices (apart from graphics) are connected with *PCIe
(Peripheral Component Interconnect Express) buses*.

### A Canonical Device

A device has to 2 important components:  
- Hardware interface: lets the OS of the connected computer control its operations  
- Internal structure: implementation specific (depends on the device) and
  responsible for abstracting the device to the system. More complex devices
  will include a simplle CPU, some general purpose memory, and other chips to
  get their job done.  

Let's examine a fake (canonical) device to understand how to make device
interaction efficient. 

### The Canonical Protocol

For this example, the canonical device interface has three registers: status
(can be read to give current status of device), command (tell the device to
perform a certain task), and data (pass or obtain data to/from the device).

Typical OS protocal with device:   
```
While (STATUS == BUSY) ; // wait until device not busy
Write data to DATA register
Write command to COMMAND register (starts device and executes command)
While (STATUS == BUSY) ; // wait until device done with request
```

Four steps:   
1. Wait until device ready to receive a command by repeatedly reading status
register (called *polling* the device).  
2. OS sends data down to data register. When the main CPU of the computer is
involve with the data movement, we refer to it as *programmed I/O (PIO)*.   
3. OS writes command to command registers. Lets the device know there data is
present and the device can work on the command. 
4. OS polls the device again to see if the protocol was successful (maybe to get
an error code). 

It is clear that polling seems inefficient.

### Lowering CPU Overhead With Interrupts

Instead of polling the device repeatedly, the OS can issue a request, put the
calling process to sleep, and context switch to another task. 

When the device is finished with the operation, it will raise a hardware
*interrupt*, causing the CPU to invoke an *interrupt handler (aka ISR)*. The CPU
switches from user mode to kernel mode and the kernel runs the handler. 

Interrupt handlers are just pieces of OS code that finish a request and wake the
process waiting for the I/O. 

If a device performs its operations quickly, then context switching to another
process only to be immediately interrupted can slow the system down due to the
expense of context switching. In this case, polling is faster.

If the speed of the device is unknown or varies, then a *hybrid* approach that
polls for a bit and then uses interrupts if the device is not yet finished may
be the best.

If a huge number of interrupts occur (like incoming packets), the OS may
*livelock* (finding itself only processing interrupts and not allowing processes
to run).  
- Solution? *Coalescing*: a device which needs to raise an interrupt first waits
  for a bit before sending an interrupt to the CPU. If there are multiple
	  interrupts at different times, they are all coalesced into a single big
	  interrupt. However, waiting too long can increase the latency of a request
	  (it is a trade-off that needs to be decided upon). 

### More Efficient Data Movement with DMA

When using PIO to transfer a large chunk of data to a device, the CPU wastes
time with such a trivial task. Solution? Use a *DMA (Direct Memory Access)*.

A DMA controller is a device (technically an I/O device itself) that can handle
transfers between main memory and devices without much CPU intervention. When a
transfer happens, the CPU switches to kernel mode and the OS tells the DMA
controller where the data lives, how much to copy, and which device to send to.
The DMA begins the transfer and the CPU switches back to user mode to continue
running processes. When the DMA engine finishes, it sends an interrupt, and the
OS knows the transfer is complete.

### Methods of Device Interaction

How does the OS communicate with devices?
 
First method: specific, privileged instructions. The OS would send instructions
like *in* or *out*, with registers specified and a port that names the device. 

Second method: *memory mapped I/O*. Hardware makes device registers available as
if they were memory locations. The hardware	makes registers available as if they
	were memory locations. To access a register, the OS issues a read to or a
	write to an address, which the hardware routes to the device instead of main
	memory.

Both are commonly used today; no specific advantage of one over the other.

### The Device Driver

How to fit devices with specific interfaces into the OS so they are as general
as possible? Example: a file system that works with USB drives, SCSI disks, etc.
Answer? Abstraction.

A low level piece of software in the OS knows in detail how a driver works. This
software is called a *device driver*. 

Example: file system  
- File systems are oblivious to specifics of which disk class it is using
- When an application issues a read, write, open, close, etc request, it does so
  to the file system. This is a system call, so the CPU switches to kernel mode.
  The OS takes this call and issues read and write requests to a *Generic Block
  Interface*. This interface then sends the request to a specific device driver.
- Cons? Using a generic block interface sometimes means that special
  capabilities in specific devices won't be used because the generic block
  interface is too abstract.
- Some special applications can issue read/write requests directly to device
  drivers without an abstract interface through the *raw interface*. 

### Case Study: Simple IDE Disk Driver

IDE disk has 4 types of registers: control, command block, status, and error.
These registers are available by reading or writing to specific I/O addresses
(on x86, using *in* and *out*). 

Protocol to interact with device: 
1. Read status register until drive is not BUSY.
2. Write sector count, logical block addresses (LBAs) of the sectors to be
   accessed, and drive number (there are two drives: master and slave) to command
   registers. 
3. Start I/O by writing READ or WRITE to command register called command status.
4. If writing, wait until status register is DRQ (drive request for data). 
5. Handle an interrupt for each sector transferred.
6. Check status register for error bit. If error bit is on, then look at error
   register for details.

Most of this protocal is found in the xv6 IDE driver, which works through 4
functions after initialization:
- `ide_rw()`, which queues a request if there are others pending. Otherwise, it
  issues a request through `ide_start_request()`. 
- `ide_start_request()`, which sends a request (and data if writing) to the
  disk. This uses `in` and `out`.
- `ide_wait_ready()` ensures the drive is ready before issuing a request to it.
- `ide_intr()` is invoked when an interrupt takes place. It reads data from the
  device (if the request is read, not write), wakes the process waiting for the
  I/O to complete and launches the next I/O if there are more requests in the
  I/O queue.


