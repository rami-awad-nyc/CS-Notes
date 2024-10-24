## Paging: Faster Translations (TLBs)

Going to physical memory for each translation is slow. We can do this with
hardware: the *translation-lookaside buffer* or the *TLB*. A TLB is part of the
MMU and is simply a cache of popular virtual-to-physical address translations. 

When a virtual address is referenced, the hardware first checks if the
translation is stored in the TLB. If not, then it goes to the page table, which
is stored in memory. 

### TLB Basic Algorithm

Assuming a simple linear page table, this is how a hardware-managed TLB works:  
1. Extract the VPN from the VA and see if TLB holds translation for VPN. If so,
we have a TLB hit. We can now extract PPN from TLB entry. 
2. If translation not in TLB (TLB miss), we look at the page table and update
the TLB with the translation. These actions are costly b/c of the extra memory
references needed to access the page table. Once TLB is updated, the translation
is found in the TLB and the memory reference is processed.

Our goal is to avoid TLB misses as often as we can. TLB hit rate is the number
of hits divided by total number of accesses. If there is a TLB miss and the TLB
is updated, the translation being then found in the TLB is not considered a hit. 

The TLB improves performance due to *spatial locality* (addresses tend to be
accessed close to one another). Only the first access to an element on a page
yields a TLB miss. As page sizes increase, TLB hit rate also increases. Typical
page size is 4KB. 

Items also tend to be referenced repeatedly in time (*temporal locality*), which
also make TLBs effective. 

### Who Handles TLB Miss?

Back in the old days, hardware would handle a TLB miss via a page-table base
register. Intel x86 architecture uses a fixed multi-level page table. Modern
architectures tend to use software-managed TLB. An exception is raised on a TLB
miss, and the CPU is switched to kernel mode. The hardware jumps to a trap
handler, which is code within the OS that handles TLB misses.

### TLB Contents

A typical TLB has many entries (32, 64, or 128) and is fully associative (a
translation can be anywhere in the TLB, not sorted or indexed or anything). A
TLB entry might look like this: VPN | PFN | other bits.

A TLB typically has a *valid bit* which says whether an entry has a valid
translation or not. Also common are *protection bits* which determine how a page
can be accessed. Other bits might be a dirty bit, address-space identifier, etc.

### TLB Issue: Context Switches

TLB contains translations only valid for the current running process. Many
approaches.

One is to flush the TLB on context switches, emptying it before running the new
process. By flushing, the valid bits for all translations are set to 0. 

However, each time a process runs, it encounters a TLB miss. Some systems reduce
this overhead by sharing TLB across context switches. They provide an *address
space identifier* (*ASID*) field in the TLB, which is like a process identifier. 

### Issue: Replacement Policy

When we install a new entry in the TLB, we must replace an old one. Which one?
We can evict the *least-recently-used* or *LRU* entry. Another common approach
is to use a random policy, which evicts a random entry.
