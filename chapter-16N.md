# note of chapter 16

1. the waste of the simple approach of using a base and bounds register pair: although the space between the stack and heap is not being used by the process, it is still taking up physical memory



**how to support a large address space with (potentially) a lot of free space between the stack and the heap?**





## 16.1 Segmentation: Generalized Base/Bounds

1. basic idea: instead of having just one base and bounds pair in our MMU, why not have a base and bounds pair per logical **segment** of the address space



## 16.2 Which Segment Are We Referring To?

1. Problems: how does it know the offset into a segment, and to which segment an address refers?



#### one explicit approach

1. basic idea: the top few bits of the virtual address shows the segment, and the rest of the virtual address shows the offset

![VAXVMS](D:\study\大二下\操作系统\ostep-hw-answer\figs\N16\VAXVMS.png)

the pseudocode of this approach in a 14-bit VA:	

```c
//get top 2 bits of 14-bit VA
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
//get offset
Offset = VirtualAddress & OFFSET_MASK
if(Offset >= Bounds[Segment])
    RaiseException(PROTECTION_FAULT)
else
    PhysAddr = Base[Segment] + Offset
    Register = AccessMemory(PhysAddr)
```

Some problems:

 	1. when using the top two bits, if we only have three segments (or less), there exist at least one unused segment
 	2. using the top so many bits to select a segment is that it limits use of the virtual address space



#### one implicit approach

1. the way hardware determines the segment: notice how the address was formed
   1. generated from the program counter: code segment
   2. based off of the stack or base pointer: stack segment
   3. else: heap segment



## 16.3 What About The Stack?

1. the difference: stack grows **backwards**
2. Solution: the hardware also needs to know which way the segment grows(a bit, that is set to 1 when growing in the positive direction, and 0 for negative)



## 16.4 Support for Sharing

1. share certain memory segments between address spaces(**code share**)
2. Solution: **protection bits**



## 16.6 OS Support

1. several issues:
   1. what should the OS do on a context switch?
      * the segment registers must be saved and restored
      * make sure to set up these registers correctly before letting the process run again
   2. OS interaction when segments grow(or shrink)
      * use malloc(): find free space for the object and return a pointer to it to the caller(in the programmer's view). However, the heap segment should grow. Using *sbrk()* system call, update the segment size register.
   3. manage free space in physical memory.
      * when a new address space is created, the OS has to find space in physical memory for its segments
      * external fragmentation: one solution is to compact physical memory by rearranging the existing segments(**expensive**); use a free-list management algorithm that tries to keep large extents of memory available for allocation(best-fit, worst-fit, first-fit, buddy algorithm)







## 16.7 Summary

leaving problems:

	1. external fragmentation
 	2. segmentation still isn't flexible enough to support fully generalized, sparse address space

