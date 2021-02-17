# note of chapter 15(vm-mechanism)

1. **hardware-based address translation(address translation)**: the hardware transforms each memory access, changing the virtual address provided to a physical address



## 15.1 Assumptions

1. the user's address space must be placed contiguously in physical memory
2. the size of the address space is not too big(less than the size of physical memory)
3. each address space is the same size



## 15.3 Dynamic(Hardware-based) Relocation

1. two registers(**base** and **bound(limit)**): place the address space anywhere we'd like in physical memory
   1. base register: transform virtual addresses into physical addresses
   2. limit register: ensure that such addresses are within the confines of the address space
2. **physical address = virtual address + base**
3. **memory management unit(MMU)**: the part of the processor that helps with address translation





## 15.4 hardware support

![support](D:\study\大二下\操作系统\ostep-hw-answer\figs\N15\hardwaresupport.png)





## 15.5 OS issues

![os](D:\study\大二下\操作系统\ostep-hw-answer\figs\N15\osissues.png)

1. notice: OS must save and restore base and limit registers in context switch in some per-process structure such as the process structure or process control block(**PCB**)



updated-LDE-boot:

![lde](D:\study\大二下\操作系统\ostep-hw-answer\figs\N15\updated-LDE.png)

updated-LDE-runtime:

![runtime](D:\study\大二下\操作系统\ostep-hw-answer\figs\N15\updated-LDE@runtime.png)