## Hard Disk Drives

Hard disk drives have been the main form of persistent data storage for decades.

### Interface

Each drive has a large number of sectors (512-byte blocks) that can each be read
or written to. The sectors are numbered 0 to *n-1* on a disk with *n* sectors.
- We can view the disk as an array of sectors where 0 to *n-1* is the address
  space of the drive

Although file systems will read or write 4KB at a time (which is many sectors),
the only guarantee drive manufacturers make is that a single sector is atomic.
If a power loss occurs, then a portion of the larger 4KB write may complete
(called a *torn write*).

### Basic Geometry

This sections discusses components of a modern disk.

A *platter* is a circular hard surface on which data is stored persistently. A
disk can have many platters. Each platter has 2 sides (top and bottom), each of
which is called a *surface*. 

The platters are all bound together around the *spindle*. When the drive is
powered on, a motor spins the platter around at a fixed rate, which is measured
in *RPM*. Modern values are between 7200 and 15000 RPM. 

Data is encoded on each surface in concentric circles of sectors. Each
concentric circle is called a *track*. 
- Concentric means that one circle is inside of another circle and so on. So in
  this case, a track is a circle inside another track and so on.

Each surface has many thousands of tracks.

The process of reading and writing from the surface is accomplished by the *disk
head*. Each surface has one disk head. All the disk heads are attached to a
single *disk arm*, which moves across the surface to position the head over the
desired track.

### Simple Disk Drive

Rotational delay: waiting for a platter to rotate so that the desired sector is
under the disk head.

Seek: the process of moving the disk head over the correct track. *Seek time*
how long it takes the disk head to move to the correct track.  4 phases in a
seek: for example of *acceleration*, then *coasting*, then *deceleration*, then
*setting* at the right track.  Settling time is usually significant.

Transfer: read/write after the head is over the correct track. 

To sum up, seek -> rotational delay -> transfer.

Other details:
* Track skew: the starting sector of each track is shifting by a few sectors
compared to previous track (track 0 starts at sector 0, track 1 starts at sector
2, etc.) so that the correct sector is rotated into position by the end of the
seek.
* Multi-zoned Disks: Since outer tracks have more space, it is a waste to have
the same number of sectors as an inner track. In a multi-zoned disk, a few
consecutive tracks are collectively known as a "zone". Each zone has a different
number of sectors per track depending on where the zone is. An outer zone will
have more sectors in each track than an inner zone.
* Cache (or track buffer): small amount of memory which the drive can use to
hold data read from or written to the disk. For reads, the drive can read all
sectors on a track and store them in cache. For writes, two choices. A *write
back* caching aka *immediate reporting* is when a write is considered complete
after data has been placed inside the cache (can be dangerous) A *write through*
caching is when a write is considered complete after data has been written to
the disk.

### Doing The Math

I/O time is the sum of three components: seek time + rotation delay time +
transfer time. The rate of I/O is the size of the transfer divided by I/O time.

Average seek time is NOT the same as full seek time (which is from one end to
the other). It is one-third of full seek time.

Also, average rotation delay time is half of how long it takes for a full
rotation.  For example, 15000 RPM is 250 RPS, meaning each rotation takes 4 ms.
Average rotation delay time is 2 ms.

The transfer time is the size of the transfer over the peak transfer rate. 

Sequential I/O workload is significantly faster than random I/O workload.

### Disk Scheduling

Which I/O request should the OS schedule next? Job of disk scheduler. The disk
scheduler tries to follow the principle of SJF (shortest job first).

SSTF: shortest seek time first. SSTF orders the queue of I/O requests by track,
picking requests on the nearest track to complete first. Problem? Drive geometry
not available to host OS. It sees an array of blocks instead. Easy fix: OS
implements nearest-block-first (NBF), which schedules the request with the
nearest block address.

Both NBF and SSTF suffer from starvation. Solution? Elevator (aka SCAN): Moves
back and forth across the disk servicing requests in order across the tracks.
C-SCAN is similar but does a sweep from inner to outer and then jumps back to
inner rather than doing another sweep from outer to inner. 

SCAN and its cousins do not represent SJF well. Solution? Shortest position time
first (SPTF). If seek time is higher than rotational delay, then SSTF and
variants are fine. However, if they are both roughly the same as on modern
devices, then SPTF is better. SPTF is difficult to implement in an OS because it
doesn't have a good idea of where track boundaries are, so SPTF is usually
performed inside a drive.

Important tasks performed by disk schedulers:
* I/O merging: if tasked to read blocks 33, 8, and 34, then the scheduler should
merge 33 and 34 requests for efficiency.
* How long should the system wait before issuing an I/O to the disk?
*Work-conserving* is immediately issuing a request to the disk for an I/O
request. Research shows that *anticipatory disk scheduling* can be better. Wait
for a bit so that a new and better request may arrive at the disk, which
increases efficiency.
