---
title: arm arm
date: 2019-05-28 13:55:44
tags:
    - arm
    - spec
categories:
    - arm
---

## Part A. Application Level Architecture
---
### A1. Introduction to the ARM Architecture
---
<!--more-->

#### A1.4 Architecture extensions
__Jazelle__
Is the Java bytecode execution extension that extended ARMv5TE to ARMv5TEJ. From
ARMv6, the architecture requires at least the trivial Jazelle implementation, but a Jazelle
implementation is still often described as a Jazelle extension.
The Virtualization Extensions require that the Jazelle implementation is the trivial Jazelle
implementation.

__ThumbEE__
Is an extension that provides the ThumbEE instruction set, a variant of the Thumb
instruction set that is designed as a target for dynamically generated code. In the original
release of the ARMv7 architecture, the ThumbEE extension was:
• A required extension to the ARMv7-A profile.
• An optional extension to the ARMv7-R profile.

From publication of issue C.a of this manual, ARM deprecates any use of ThumbEE
instructions. However, ARMv7-A implementations must continue to include ThumbEE
support, for backwards compatibility.

__Floating-point__
Is a floating-point coprocessor extension to the instruction set architectures. For historic
reasons, the Floating-point Extension is also called the VFP Extension. 

__Advanced SIMD__
Is an instruction set extension that provides Single Instruction Multiple Data (SIMD)
integer and single-precision floating-point vector operations on doubleword and quadword
registers.	

### A1.4.2 Architecture extensions
This manual also describes the following extensions to the ARMv7 architecture:
__Security Extensions__

__Multiprocessing Extensions__
Are an OPTIONAL set of extensions to the ARMv7-A and ARMv7-R profiles, that provides a set of
features that enhance multiprocessing functionality.

__Large Physical Address Extension__
Is an OPTIONAL extension to VMSAv7 that provides an address translation system supporting
physical addresses of up to 40 bits at a fine grain of translation.
The Large Physical Address Extension requires implementation of the Multiprocessing Extensions.

__Virtualization Extensions__
Are an OPTIONAL set of extensions to VMSAv7 that provides hardware support for virtualizing the
Non-secure state of a VMSAv7 implementation. This supports system use of a virtual machine
monitor, also called a hypervisor.

__Generic Timer Extension__
Is an OPTIONAL extension to any ARMv7-A or ARMv7-R, that provides a system timer, and a
low-latency register interface to it.

__Performance Monitors Extension__
The ARMv7 architecture:
• reserves CP15 register space for IMPLEMENTATION DEFINED performance monitors
• defines a recommended performance monitors implementation.

---
### A2. Application Level Programmers’ Model
#### A2.1 About the Application level programmers’ model
Depending on the implemented architecture extensions, the architecture supports multiple levels of execution
privilege, that number upwards from PL0, where PL0 is the lowest privilege level and is often described as
unprivileged.

When an operating system supports execution at both PL1 and PL0, an application usually runs unprivileged. This:
• permits the operating system to __allocate system resources__ to an application in a unique or shared manner
• provides __a degree of protection from other processes and tasks__, and so helps protect the operating system
from malfunctioning applications.

#### A2.2 ARM core data types and arithmetic
All ARMv7-A and ARMv7-R processors support the following data types in memory:
Byte           8 bits
Halfword       16 bits
Word           32 bits
Doubleword     64 bits.

Processor registers are 32 bits in size. The instruction set contains instructions supporting the following data types
held in registers:
• 32-bit pointers
• unsigned or signed 32-bit integers
• unsigned 16-bit or 8-bit integers, held in zero-extended form
• signed 16-bit or 8-bit integers, held in sign-extended form
• two 16-bit integers packed into a register
• four 8-bit integers packed into a register
• unsigned or signed 64-bit integers held in two registers.

#### A2.3 ARM core registers
In the application level view, an ARM processor has:
• thirteen general-purpose 32-bit registers, R0 to R12
• three 32-bit registers with special uses, SP, LR, and PC, that can be described as R13 to R15.
The special registers are:
__SP, the stack pointer__
The processor uses SP as a pointer to the active stack.
In the Thumb instruction set, most instructions cannot access SP. The only instructions that can
access SP are those designed to use SP as a stack pointer.
The ARM instruction set provides more general access to the SP, and it can be used as a
general-purpose register. However, ARM deprecates the use of SP for any purpose other than as a
stack pointer.Software can refer to SP as R13.

__LR, the link register__
The link register is a special register that can hold return link information. Some cases described in
this manual require this use of the LR. When software does not require the LR for linking, it can use
it for other purposes. It can refer to LR as R14.

__PC, the program counter__
• When executing an ARM instruction, PC reads as the address of the current instruction plus 8.（PC始终指向你要取的指令的地址。ARM三级流水线，下一条指令是包含了预取指令，执行指令）
• When executing a Thumb instruction, PC reads as the address of the current instruction plus 4.
• Writing an address to PC causes a branch to that address.
Most Thumb instructions cannot access PC.
The ARM instruction set provides more general access to the PC, and many ARM instructions can
use the PC as a general-purpose register. However, ARM deprecates the use of PC for any purpose
other than as the program counter. Software can refer to PC as R15.

##### A2.3.1 Writing to the PC
In ARMv7, many data-processing instructions can write to the PC. Writes to the PC are handled as follows:
• The B, BL, CBNZ, CBZ, CHKA, HB, HBL, HBLP, HBP, TBB, and TBH instructions remain in the same instruction set state
and branch to the value written to the PC.
The definition of each of these instructions ensures that the value written to the PC is correctly aligned for
the current instruction set state.

• The BLX (immediate) instruction switches between ARM and Thumb states and branches to the value written
to the PC. Its definition ensures that the value written to the PC is correctly aligned for the new instruction
set state.

• The following instructions write a value to the PC, treating that value as an interworking address to branch
to, with low-order bits that determine the new instruction set state:
	— BLX (register), BX, and BXJ
	— LDR instructions with <Rt> equal to the PC
	— POP and all forms of LDM except LDM (exception return), when the register list includes the PC
	— in ARM state only, ADC, ADD, ADR, AND, ASR (immediate), BIC, EOR, LSL (immediate), LSR (immediate), MOV,
		MVN, ORR, ROR (immediate), RRX, RSB, RSC, SBC, and SUB instructions with <Rd> equal to the PC and without
		flag-setting specified.

#### A2.4 The Application Program Status Register (APSR)
Fixme [APSR] Page49

• Bits that can be set by many instructions:
— The Condition flags:
	N, bit[31] Negative condition flag. Set to bit[31] of the result of the instruction. If the result is
	regarded as a two's complement signed integer, then the processor sets N to 1 if the result
	is negative, and sets N to 0 if it is positive or zero.
	Z, bit[30] Zero condition flag. Set to 1 if the result of the instruction is zero, and to 0 otherwise. A
	result of zero often indicates an equal result from a comparison.
	C, bit[29] Carry condition flag. Set to 1 if the instruction results in a carry condition, for example an
	unsigned overflow on an addition.
	V, bit[28] Overflow condition flag. Set to 1 if the instruction results in an overflow condition, for
	example a signed overflow on an addition.

— The Overflow or saturation flag:
	Q, bit[27] Set to 1 to indicate overflow or saturation occurred in some instructions, normally related
	to digital signal processing (DSP). For more information, see Pseudocode details of
	saturation on page A2-44.

— The Greater than or Equal flags:
	GE[3:0], bits[19:16]
	The instructions described in Parallel addition and subtraction instructions on
	page A4-171 update these flags to indicate the results from individual bytes or halfwords
	of the operation. These flags can control a later SEL instruction. For more information, see
	SEL on page A8-602.

In ARMv7-A and ARMv7-R, the APSR is the same register as the CPSR, but the APSR must be used only to access
the N, Z, C, V, Q, and GE[3:0] bits.

#### A2.5 Execution state registers
The execution state registers modify the execution of instructions. They control:
• Whether instructions are interpreted as Thumb instructions, ARM instructions, ThumbEE instructions, or
Java bytecodes. For more information, see Instruction set state register, ISETSTATE.

• In Thumb state and ThumbEE state only, the condition codes that apply to the next one to four instructions.
For more information, see IT block state register, ITSTATE on page A2-51.

• Whether data is interpreted as big-endian or little-endian. For more information, see Endianness mapping
register, ENDIANSTATE on page A2-53.

In ARMv7-A and ARMv7-R, the execution state registers are part of the Current Program Status Register. For more
information, see Program Status Registers (PSRs) on page B1-1147.

##### A2.5.1 Instruction set state register, ISETSTATE
The instruction set state register, ISETSTATE, format is:
The J bit and the T bit determine the current instruction set state for the processor. Table A2-1 shows the encoding
of these bits.

Fixme [Table A2-1 J and T bit encoding in ISETSTATE]Page50

##### A2.5.2 IT block state register, ITSTATE
Fixme [Table A2-2 Effect of IT execution state bits]Page52

##### A2.5.3 Endianness mapping register, ENDIANSTATE
Fixme [Table A2-3 ENDIANSTATE encoding of endianness]Page53


#### A2.6 Advanced SIMD and Floating-point Extensions
Advanced SIMD and Floating-point (VFP) are two OPTIONAL extensions to ARMv7.

The Advanced SIMD Extension performs packed Single Instruction Multiple Data (SIMD) operations, either
integer or single-precision floating-point.

....


#### A2.11 Jazelle direct bytecode execution support
The Jazelle extension provides architectural support for hardware acceleration of bytecode execution by a Java Virtual
Machine (JVM).

These requirements for the Jazelle extension mean a JVM can be written to both:
• function correctly on all processors that include a Jazelle extension implementation
• automatically take advantage of the accelerated bytecode execution provided by a processor that includes a
non-trivial implementation.

The required features of a non-trivial implementation are:
• provision of the Jazelle state
• a new instruction, BXJ, to enter Jazelle state
• system support that enables an operating system to regulate the use of the Jazelle extension hardware
• system support that enables a JVM to configure the Jazelle extension hardware to its specific needs.

...

### A3 Application Level Memory Model

...

#### A3.2 Alignment support
Instructions in the ARM architecture are aligned as follows:
• ARM instructions are word-aligned
• Thumb and ThumbEE instructions are halfword-aligned
• Java bytecodes are byte-aligned.
In the ARMv7 architecture, some load and store instructions support unaligned data accesses, as described in
Unaligned data access.

##### A3.2.1 Unaligned data access
An ARMv7 implementation must support unaligned data accesses to Normal memory by some load and store
instructions. __Software can set the SCTLR.A bit to control whether a misaligned access to Normal memory by one of these instructions causes an Alignment fault Data Abort exception.__

__Unaligned access operations must not be used for accessing memory-mapped registers in a Device or Strongly-ordered memory region.__

Fixme [Table A3-1 Alignment requirements of load/store instructions]  page108

#### A3.3 Endian support
Data support big-endian or little-endian.

##### A3.3.1 Instruction endianness
__In ARMv7-A, the mapping of instruction memory is always little-endian__. In ARMv7-R, instruction endianness can
be controlled at the system level, In ARMv7-A, the mapping of instruction memory is always little-endian. In ARMv7-R, instruction endianness can be controlled at the system level, see Instruction endianness static configuration, ARMv7-R only on page A3-112.

##### A3.3.2 Element size and endianness

Fixme [Table A3-2 Element size of load/store instructions] page112

##### A3.3.4 Endianness in Advanced SIMD
Advanced SIMD element load/store instructions transfer vectors of elements between memory and the Advanced
SIMD register bank.An instruction specifies both the length of the transfer and the size of the data elements being
transferred. This information is used by the processor to load and store data correctly in both big-endian and
little-endian systems.

处理器根据提供的信息，能保证在Advanced SIMD register bank 中的数据是一样的，不论它是否是大端，小端
Fixme [Figure A3-2 Advanced SIMD byte order example] page113

#### A3.4 Synchronization and semaphores
In architecture versions before ARMv6, support for the synchronization of shared memory depends on the SWP and
SWPB instructions.These are read-locked-write operations that swap register contents with memory, and are
described in SWP, SWPB on page A8-722. These instructions support basic busy/free semaphore mechanisms, but
do not support mechanisms that require calculation to be performed on the semaphore between the read and write
phases.

ARMv7 extends support for
this mechanism, and provides the following synchronization primitives in the ARM and Thumb instruction sets:
• Load-Exclusives:
	— LDREX, see LDREX on page A8-432
	— LDREXB, see LDREXB on page A8-434
	— LDREXD, see LDREXD on page A8-436
	— LDREXH, see LDREXH on page A8-438
• Store-Exclusives:
	— STREX, see STREX on page A8-690
	— STREXB, see STREXB on page A8-692
	— STREXD, see STREXD on page A8-694
	— STREXH, see STREXH on page A8-696
• Clear-Exclusive, CLREX, see CLREX on page A8-360.

Note
• ARM strongly recommends that all software uses the synchronization primitives described in this section,
rather than SWP or SWPB.

##### A3.4.1 Exclusive access instructions and Non-shareable memory regions
For memory regions that do not have the Shareable attribute, the exclusive access instructions __rely on a local monitor that tags any address__ from which the processor executes a Load-Exclusive. Any non-aborted attempt by the
same processor to use a Store-Exclusive to modify any address is guaranteed to clear the tag.

A Load-Exclusive performs a load from memory, and:
• the executing processor tags the physical memory address for exclusive access
• the local monitor of the executing processor transitions to the Exclusive Access state.

A Store-Exclusive performs a conditional store to memory, that depends on the state of the local monitor:
__If the local monitor is in the Exclusive Access state__
	• If the address of the Store-Exclusive is the same as the address that has been tagged in the
	monitor by an earlier Load-Exclusive, then the store occurs, otherwise it is IMPLEMENTATION
	DEFINED whether the store occurs.
	• A status value is returned to a register:
		— if the store took place the status value is 0
		— otherwise, the status value is 1.
	• The local monitor of the executing processor transitions to the Open Access state.
__If the local monitor is in the Open Access state__
	• no store takes place
	• a status value of 1 is returned to a register.
	• the local monitor remains in the Open Access state.

Fixme [Figure A3-3 Local monitor state machine diagram]Page 116	

Fixme [Table A3-3 Effect of Exclusive instructions and write operations on the local monitor]Page 116


##### A3.4.2 Exclusive access instructions and Shareable memory regions
For memory regions that have the Shareable attribute, exclusive access instructions rely on:
• __A local monitor for each processor in the system__, that tags any address from which the processor executes a
Load-Exclusive. The local monitor can ignore accesses from other processors in the system.

• __A global monitor that tags a physical address as exclusive access for a particular processor__. This tag is used
later to determine whether a Store-Exclusive to that address that has not been failed by the local monitor can
occur. Any successful write to the tagged address by any other observer in the shareability domain of the
memory location is guaranteed to clear the tag. For each processor in the system, the global monitor:
	— can hold at least one tagged address
	— maintains a state machine for each tagged address it can hold.

__Operation of the global monitor__
A Load-Exclusive from Shareable memory performs a load from memory, and causes the __physical address of the access to be tagged as exclusive access for the requesting processor.__ This access also causes the exclusive access tag to be removed from any other physical address that has been tagged by the requesting processor.

The global monitor only supports a single outstanding exclusive access to Shareable memory per processor. A
Load-Exclusive by one processor has no effect on the global monitor state for any other processor.

Fixme [Table A3-4 Effect of load/store operations on global monitor for processor(n)] Page120


##### A3.4.3 Tagging and the size of the tagged memory block
Tagged_address = Memory_address[31:a]
The value of a in this assignment is IMPLEMENTATION DEFINED, between a minimum value of 3 and a maximum
value of 11. For example, in an implementation where a is 4, a successful LDREX of address 0x000341B4 gives a tag
value of bits[31:4] of the address, giving 0x000341B. This means that the four words of memory from 0x000341B0 to
0x000341BF are tagged for exclusive access.

The size of the tagged memory block is called the Exclusives Reservation Granule. The Exclusives Reservation
Granule is IMPLEMENTATION DEFINED in the range 2-512 words:
• 2 words in an implementation where a is 3
• 512 words in an implementation where a is 11.

##### A3.4.4 Context switch support
After a context switch, software must ensure that the local monitor is in the Open Access state. This requires it to
either:
• execute a CLREX instruction
• execute a dummy STREX to a memory address allocated for this purpose.

Note:
Using a dummy STREX for this purpose is backwards-compatible with the ARMv6 implementation of the
exclusive operations. The CLREX instruction is introduced in ARMv6K.

##### A3.4.5 Load-Exclusive and Store-Exclusive usage restrictions
The Load-Exclusive and Store-Exclusive instructions are intended to work together, as a pair, for example a
LDREX/STREX pair or a LDREXB/STREXB pair.

• An implementation of the Load-Exclusive and Store-Exclusive instructions can require that, in any thread of
execution, the transaction size of a Store-Exclusive is the same as the transaction size of the preceding
Load-Exclusive executed in that thread.

• An implementation might clear an exclusive monitor between the LDREX and the STREX, without any
application-related cause. Software written for such an implementation must, in any single thread of execution, __avoid having any explicit memory accesses, System control register updates, or cache maintenance operations between the LDREX instruction and the associated STREX instruction.__

• In some implementations, an access to Strongly-ordered or Device memory might clear the exclusive
monitor. Therefore, __software must not place a load or a store to Strongly-ordered or Device memory between an LDREX and an STREX in a single thread of execution.__

• Implementations can benefit from keeping the LDREX and STREX operations close together in a single thread of
execution. This minimizes the likelihood of the exclusive monitor state being cleared between the LDREX
instruction and the STREX instruction. Therefore, __for best performance, ARM strongly recommends a limit of 128 bytes between LDREX and STREX instructions in a single thread of execution.__

• After taking a Data Abort exception, the state of the exclusive monitors is UNKNOWN. Therefore ARM
strongly recommends that the abort handling software performs a CLREX instruction, or a dummy STREX
instruction, to clear the monitor state.

• The effect of a data or unified cache invalidate, cache clean, or cache clean and invalidate instruction on a
local or global exclusive monitor that is in the Exclusive Access state is UNPREDICTABLE. Execution of the
instruction might clear the monitor, or it might leave it in the Exclusive Access state.

• For the memory location being accessed by a LoadExcl/StoreExcl pair, if the memory attributes for the
LoadExcl instruction differ from the memory attributes for the StoreExcl instruction, behavior is
UNPREDICTABLE.

##### A3.4.6 Semaphores
The Swap (SWP) and Swap Byte (SWPB) instructions must be used with care to ensure that expected behavior is
observed.

Note
From ARMv6, ARM deprecates use of the Swap and Swap Byte instructions, and strongly recommends that all new
software uses the Load-Exclusive and Store-Exclusive synchronization primitives

##### A3.4.7 Synchronization primitives and the memory order model
The synchronization primitives follow the memory order model of the memory type accessed by the instructions.
For this reason:
• Portable software for claiming a spin-lock must include a Data Memory Barrier (DMB) operation, performed
by a DMB instruction, between claiming the spin-lock and making any access that makes use of the spin-lock.
• Portable software for releasing a spin-lock must include a DMB instruction before writing to clear the spin-lock.
This requirement applies to software using:
• the Load-Exclusive/Store-Exclusive instruction pairs, for example LDREX/STREX
• the deprecated synchronization primitives, SWP/SWPB.

[ISB > DSB > DMB](https://blog.csdn.net/wangbinyantai/article/details/78986974)

##### A3.4.8 Use of WFE and SEV instructions by spin-locks
ARMv7 and ARMv6K provide Wait For Event and Send Event instructions, WFE and SEV, that can assist with
reducing power consumption and bus contention caused by processors repeatedly attempting to obtain a spin-lock.
These instructions can be used at the application level, but a complete understanding of what they do depends on
system level understanding of exceptions.

#### A3.5 Memory types and attributes and the memory order model
##### A3.5.1 Memory types
exclusive memory types:
• Normal
• Device
• Strongly-ordered.

##### A3.5.2 Summary of ARMv7 memory attributes

__Shareability__
Applies only to Normal memory, and __to Device memory in an implementation that does not include the Large Physical Address Extension.__ In an implementation that includes the Large Physical
Address Extension, Device memory is always Outer Shareable,
When it is possible to assign a shareability attribute to Device memory, ARM deprecates assigning
any attribute other than Shareable or Outer Shareable, see Shareable attribute for Device memory
regions on page A3-137
Whether an ARMv7 implementation distinguishes between Inner Shareable and Outer Shareable
memory is IMPLEMENTATION DEFINED.

__Cacheability__
Applies only to Normal memory, and can be defined independently for Inner and Outer cache
regions. Some cacheability attributes can be complemented by a cache allocation hint. This is an
indication to the memory system of whether allocating a value to a cache is likely to improve
performance. 

Fixme [Table A3-5 Memory attribute summary] page127

##### A3.5.3 Atomicity in the ARM architecture
Atomicity is a feature of memory accesses, described as atomic accesses. The ARM architecture description refers
to two types of atomicity, defined in:
• Single-copy atomicity
• Multi-copy atomicity on page A3-130.

__Single-copy atomicity__
In ARMv7, the single-copy atomic processor accesses are:
• All byte accesses.
• All halfword accesses to halfword-aligned locations.
• All word accesses to word-aligned locations.
• Memory accesses caused by a LDREXD/STREXD to a doubleword-aligned location for which the STREXD succeeds
cause single-copy atomic updates of the doubleword being accessed.
Note
The way to atomically load two 32-bit quantities is to perform a LDREXD/STREXD sequence, reading and writing
the same value, for which the STREXD succeeds, and use the read values

__Multi-copy atomicity__
In a multiprocessing system, writes to a memory location are multi-copy atomic

Writes to Normal memory are not multi-copy atomic. (有缓存机制以及SNOOP等)
All writes to Device and Strongly-ordered memory that are single-copy atomic are also multi-copy atomic.

##### A3.5.4 Concurrent modification and execution of instructions
The ARMv7 architecture limits the set of instructions that can be executed by one thread of execution as they are
being modified by another thread of execution without requiring explicit synchronization.

Except for the instructions identified in this section, the effect of the concurrent modification and execution of an
instruction is UNPREDICTABLE. （无条件跳转等命令，不受这个限制）


__In the Thumb instruction set__
The 16-bit encodings of the B, NOP, BKPT, and SVC instructions

__In the ARM instruction set__
The B, BL, NOP, BKPT, SVC, HVC, and SMC instructions.

##### A3.5.5 Normal memory
Accesses to normal memory region are idempotent, meaning that they exhibit the following properties:
• read accesses can be repeated with no side-effects
• repeated read accesses return the last value written to the resource being read
• read accesses can fetch additional memory locations with no side-effects
• write accesses can be repeated with no side-effects in the following cases:
	— if the contents of the location accessed are unchanged between the repeated writes
	— as the result of an exception, as described in this section
• unaligned accesses can be supported
• accesses can be merged before accessing the target memory system.

Normal memory can be read/write or read-only, and a Normal memory region is defined as being either Shareable
or Non-shareable.

__Non-shareable Normal memory__
For a Normal memory region, the Non-shareable attribute identifies Normal memory that is likely to be accessed
only by a single processor.

__Shareable, Inner Shareable, and Outer Shareable Normal memory__
For Normal memory, the Shareable and Outer Shareable memory attributes describe Normal memory that is
expected to __be accessed by multiple processors or other system masters:__
• In a VMSA implementation, Normal memory that has the Shareable attribute but not the Outer Shareable
attribute assigned is described as having the Inner Shareable attribute.
• In a PMSA implementation, no distinction is made between Inner Shareable and Outer Shareable Normal
memory.

VMSA: Virtual Memory System Architecture
PMSA: Protected Memory System Architecture

__Write-Through Cacheable, Write-Back Cacheable and Non-cacheable Normal memory__
The cacheability attributes provide a mechanism of coherency control with observers that lie outside the shareability
domain of a region of memory.
• Write-Through Cacheable
• Write-Back Cacheable
• Non-cacheable.

The cacheability attributes provide a mechanism of coherency control with observers that lie outside the shareability
domain of a region of memory. In some cases, the use of Write-Through Cacheable or Non-cacheable regions of
memory might provide a better mechanism for controlling coherency than the use of hardware coherency
mechanisms or the use of cache maintenance routines.

##### A3.5.6 Device and Strongly-ordered memory
Examples of memory regions normally marked as being Device or Strongly-ordered memory are Memory-mapped
peripherals and I/O locations. __Address locations marked as Device or Strongly-ordered are never held in a cache.__

The architecture permits an Advanced SIMD element or structure load instruction to access bytes in Device or
Strongly-ordered memory.

__The architecture does not permit unaligned accesses to Strongly-ordered or Device memory.__

###### Shareable attribute for Device memory regions
In an implementation that does not include the Large Physical Address Extension, Device memory regions can be
given the Shareable attribute. When a Device memory region is give the Shareable attribute it can also be given the
Outer Shareable attribute. This means that a region of Device memory can be described as one of:
• Outer Shareable Device memory
• Inner Shareable Device memory
• Non-shareable Device memory.


ARM deprecates the marking of Device memory with a shareability attribute other than Outer Shareable or
Shareable. This means __ARM strongly recommends that Device memory is never assigned a shareability attribute of Non-shareable or Inner Shareable.__

###### Device and Strongly-ordered memory shareability, Large Physical Address Extension
In an implementation that includes the Large Physical Address Extension, the Long-descriptor translation table
format does not distinguish between Shareable and Non-shareable Device memory.

In an implementation that includes the Large Physical Address Extension and is using the Short-descriptor
translation table format:
• An address-based cache maintenance operation for an addresses in a region with the Strongly-ordered or
Device memory type applies to all processors in the same Outer Shareable domain, regardless of any
shareability attributes applied to the region.
• Device memory transactions to a single peripheral must not be reordered, regardless of any shareability
attributes that are applied to the corresponding Device memory region.
Any single peripheral has an IMPLEMENTATION DEFINED size of not less than 1KB.

##### A3.5.7 Memory access restrictions
The following restrictions apply to memory accesses:
• For accesses to any two bytes, p and q, that are generated by the same instruction:
	— The bytes p and q must have the same memory type and shareability attributes
	— Except for possible differences in the cache allocation hints, ARM deprecates having different
	cacheability attributes for the bytes p and q.

• Unaligned data access on page A3-108 identifies the instructions that can make an unaligned memory
access,If such an access is to Device or Strongly-ordered memory then:
	— if the implementation does not include the Virtualization Extensions, the effect is UNPREDICTABLE
	— if the implementation includes the Virtualization Extensions, the access generates an Alignment fault

• __The accesses of an instruction that causes multiple accesses to Device or Strongly-ordered memory must not cross a 4KB address boundary__

• __Any instruction fetch must access only Normal memory.__ If it accesses Device or Strongly-ordered memory,
the result is UNPREDICTABLE.


##### A3.5.8 The effect of the Security Extensions
The Security Extensions can be included as part of an ARMv7-A implementation, with a VMSA. They provide two
distinct 4GByte virtual memory spaces:
• a Secure virtual memory space
• a Non-secure virtual memory space.
The Secure virtual memory space is accessed by memory accesses in the Secure state, and the Non-secure virtual
memory space is accessed by memory accesses in the Non-secure state.
By providing different virtual memory spaces, the Security Extensions permit memory accesses made from the
Non-secure state to be distinguished from those made from the Secure state.

#### A3.6 Access rights
##### A3.6.1 Processor privilege levels, execution privilege, and access privilege
ARMv7 architecture defines different levels of execution privilege:
• in Secure state, the privilege levels are PL1 and PL0
• in Non-secure state, the privilege levels are PL2, PL1, and PL0.

__PL0__
The privilege level of application software, that executes in User mode. Therefore, software
executed in User mode is described as unprivileged software. This software cannot access some
features of the architecture. In particular, it cannot change many of the configuration settings.
Software executing at PL0 makes only unprivileged memory accesses.

__PL1__
Software execution in all modes other than User mode and Hyp mode is at PL1. Normally, operating
system software executes at PL1. Software executing at PL1 can access all features of the
architecture, and can change the configuration settings for those features, except for some features
added by the Virtualization Extensions that are only accessible at PL2.
Note
In many implementation models, system software is unaware of the PL2 level of privilege, and of
whether the implementation includes the Virtualization Extensions.
The PL1 modes refers to all the modes other than User mode and Hyp mode.
Software executing at PL1 makes privileged memory accesses by default, but can also make
unprivileged accesses.

__PL2__
Software executing in Hyp mode executes at PL2.
Software executing at PL2 can perform all of the operations accessible at PL1, and can access some
additional functionality.
Hyp mode is normally used by a hypervisor, that controls, and can switch between, Guest OSs, that
execute at PL1.

Hyp mode is implemented only as part of the Virtualization Extensions, and only in Non-secure
state. This means that:
• implementations that do not include the Virtualization Extensions have only two privilege
levels, PL0 and PL1
• execution in Secure state has only two privilege levels, PL0 and PL1

#### A3.8 Memory access order

##### A3.8.1 Reads and writes
The following can cause memory accesses that are not explicit:
• instruction fetches
• cache loads and write-backs
• translation table walks.

###### Reads
Reads are defined as memory operations that have the semantics of a load.
The memory accesses of the following instructions are reads:
• LDR, LDRB, LDRH, LDRSB, and LDRSH.
• LDRT, LDRBT, LDRHT, LDRSBT, and LDRSHT.
• LDREX, LDREXB, LDREXD, and LDREXH.
• LDM, LDRD, POP, and RFE.
• LDC, LDC2, VLDM, VLDR, VLD1, VLD2, VLD3, VLD4, and VPOP.
• The return of status values by STREX, STREXB, STREXD, and STREXH.
• SWP and SWPB. These instructions are available only in the ARM instruction set.
• TBB and TBH. These instructions are available only in the Thumb instruction set.
Hardware-accelerated opcode execution by the Jazelle extension can cause a number of reads to occur, according
to the state of the operand stack and the implementation of the Jazelle hardware acceleration.

###### Writes
Writes are defined as memory operations that have the semantics of a store.
The memory accesses of the following instructions are Writes:
• STR, STRB, and STRH.
• STRT, STRBT, and STRHT.
• STREX, STREXB, STREXD, and STREXH.
• STM, STRD, PUSH, and SRS.
• STC, STC2, VPUSH, VSTM, VSTR, VST1, VST2, VST3, and VST4.
• SWP and SWPB. These instructions are available only in the ARM instruction set.
Hardware-accelerated opcode execution by the Jazelle extension can cause a number of writes to occur, according
to the state of the operand stack and the implementation of the Jazelle hardware acceleration.

###### Synchronization primitives
Synchronization primitives must ensure correct operation of system semaphores in the memory order model. They are the following instructions:
• LDREX, STREX, LDREXB, STREXB, LDREXD, STREXD, LDREXH, STREXH.
• SWP, SWPB. From ARMv6, ARM deprecates the use of these instructions.

##### A3.8.3 Memory barriers
Memory barrier is the general term applied to an instruction, or sequence of instructions, that forces synchronization
events by a processor with respect to retiring load/store instructions.

The ARM architecture defines a number of memory barriers that provide a range of functionality, including:
• ordering of load/store instructions
• completion of load/store instructions
• context synchronization.

In ARMv7 the memory barriers are provided as instructions that are available in the ARM and Thumb
instruction sets, and in ARMv6 the memory barriers are performed by CP15 register writes. The three memory
barriers are:
• Data Memory Barrier, see Data Memory Barrier (DMB) on page A3-152
• Data Synchronization Barrier, see Data Synchronization Barrier (DSB) on page A3-153
• Instruction Synchronization Barrier, see Instruction Synchronization Barrier (ISB) on page A3-153.

###### Data Memory Barrier (DMB)
The DMB instruction is a data memory barrier. __DMB only affects memory accesses and data and unified cache maintenance operations,  It has no effect on the ordering of any other instructions executing on the processor.__

###### Data Synchronization Barrier (DSB)
The DSB instruction is a special memory barrier, that synchronizes the execution stream with memory accesses. __In addition, no instruction that appears in program order after the DSB instruction can execute until the DSB completes.__

###### Instruction Synchronization Barrier (ISB)
__An ISB instruction flushes the pipeline in the processor, so that all instructions that come after the ISB instruction in program order are fetched from cache or memory only after the ISB instruction has completed.__

Using an ISB ensures that the effects of context-changing operations executed before the ISB are visible to the instructions fetched after the ISB instruction. 

Examples of context-changing operations that require the insertion of an ISB instruction to ensure
the effects of the operation are visible to instructions fetched after the ISB instruction are:
• completed cache, TLB, and branch predictor maintenance operations
• changes to system control registers. 

#### A3.9 Caches and memory hierarchy

##### A3.9.1 Introduction to caches
A cache is a block of high-speed memory that contains a number of entries, each consisting of:
• main memory address information, commonly called a tag
• the associated data

##### A3.9.2 Memory hierarchy
Memory close to a processor has very low latency, but is limited in size and expensive to implement.To optimize overall
performance, an ARMv7 memory system can include multiple levels of cache in a hierarchical memory system. 

Fixme[Figure A3-6 Multiple levels of cache in a memory hierarchy] page 157

##### A3.9.4 Preloading caches
The ARM architecture provides memory system hints PLD (Preload Data), PLDW (Preload Data with intent to write),
and PLI (Preload Instruction) to permit software to communicate the expected use of memory locations to the
hardware. 

### A4. The Instruction Sets
#### A4.1 About the instruction sets
ARMv7 contains two main instruction sets, the ARM and Thumb instruction sets. 

The ARM and Thumb instruction sets can interwork freely, that is, different procedures can be compiled or assembled to different instruction sets, and still be able to call each other efficiently.

__ThumbEE__ is a variant of the Thumb instruction set that is designed as a target for __dynamically generated code.__
However, it cannot interwork freely with the ARM and Thumb instruction sets.

The two instruction sets differ in how instructions are encoded:
• Thumb instructions are either 16-bit or 32-bit, and are aligned on __a two-byte boundary__. 16-bit and 32-bit
instructions can be intermixed freely. Many common operations are most efficiently executed using 16-bit
instructions. However:
	— Most 16-bit instructions can only access the first eight of the ARM core registers, R0-R7. These are
	called the low registers. A small number of 16-bit instructions can also access the high registers,
	R8-R15.
	— Many operations that would require two or more 16-bit instructions can be more efficiently executed
	with a single 32-bit instruction.
	— All 32-bit instructions can access all of the ARM core registers, R0-R15.

• ARM instructions are always 32-bit, and are aligned on a four-byte boundary

##### A4.1.1 Changing between Thumb state and ARM state

###### Thumb to ARM state
A processor in Thumb state can enter ARM state by executing any of the following instructions: __BX, BLX, or an LDR or LDM that loads the PC.__

###### ARM to Thumb state
A processor in ARM state can enter Thumb state by executing any of the same instructions.

In ARMv7, a processor in ARM state can also enter Thumb state by executing an ADC, ADD, AND, ASR, BIC, EOR, LSL,
LSR, MOV, MVN, ORR, ROR, RRX, RSB, RSC, SBC, or SUB instruction that __has the PC as destination register and does not set the condition flags.__

###### others
The target instruction set is either encoded directly in the instruction (for the immediate offset version of BLX), or is
held as bit[0] of an interworking address. For details, see the description of the BXWritePC() function in Pseudocode
details of operations on ARM core registers on page A2-47.
bit[0]    1 thumb      thumb is 2 bytes boundary but ARM some support Jazella extension
bit[1:0] 00 arm        arm is 4 bytes boundary
bit[1:0] 10 unknown

Exception entries and returns can also change between ARM and Thumb states. For details see Exception handling
on page B1-1165.

##### A4.1.2 Conditional execution
In the ARM and Thumb instruction sets, most instructions can be conditionally executed

In the ARM instruction set,  has its normal effect on the programmers’ model operation, memory and coprocessors if the N, Z, C and V condition flags in the APSR satisfy.

If the flags do not satisfy this condition, the instruction acts as a NOP.

In the Thumb instruction set, different mechanisms control conditional execution:
• For the following Thumb encodings, conditional execution is controlled in a similar way to the ARM
instructions:
	— A 16-bit conditional branch instruction encoding, with a branch range of –256 to +254 bytes. Before
	ARMv6T2, this was the only mechanism for conditional execution in Thumb code.
	— A 32-bit conditional branch instruction encoding, with a branch range of approximately ±1MB.

• The CBZ and CBNZ instructions, Compare and Branch on Zero and Compare and Branch on Nonzero, are 16-bit
conditional instructions with a branch range of +4 to +130 bytes.	

• The 16-bit If-Then instruction makes up to four following instructions conditional, and can make most other
Thumb instructions conditional. For details see IT on page A8-390. 

#### A4.3 Branch instructions
Fixme [Table A4-1 Branch instructions] Page164

#### A4.4 Data-processing instructions
##### A4.4.1 Standard data-processing instructions
Fixme [Table A4-2 Standard data-processing instructions] Page166

##### A4.4.2 Shift instructions
Fixme [Table A4-3 Shift instructions] Page167

##### A4.4.3 Multiply instructions
These instructions can operate on signed or unsigned quantities. In some types of operation, the results are same
whether the operands are signed or unsigned.

Fixme [Table A4-4 General multiply instructions] Page167

Fixme [Table A4-5 Signed multiply instructions] Page168

Fixme [Table A4-6 Unsigned multiply instructions] Page168

##### A4.4.4 Saturating instructions
饱和指令： 将超出unsigned, signed 的值限制到本身支持的最大值或最小值

##### A4.4.5 Saturating addition and subtraction instructions

##### A4.4.6 Packing and unpacking instructions
扩展指令： 将[半]字节扩展到[有/无]32位

##### A4.4.7 Parallel addition and subtraction instructions
These instructions perform additions and subtractions on the values of two registers and write the result to a
destination register, treating the register values as sets of two halfwords or four bytes. That is, they perform SIMD
additions or subtractions on the registers. 

##### A4.4.8 Divide instructions

For descriptions of the instructions see:
• SDIV on page A8-600
• UDIV on page A8-760

In the ARMv7-R profile, the SCTLR.DZ bit enables divide by zero fault detection:
SCTLR.DZ == 0 Divide-by-zero returns a zero result.
SCTLR.DZ == 1 SDIV and UDIV generate an Undefined Instruction exception on a divide-by-zero.
The SCTLR.DZ bit is cleared to zero on reset

##### A4.4.9 Miscellaneous data-processing instructions
Fixme [Table A4-11 Miscellaneous data-processing instructions] page173

#### A4.5 Status register access instructions

The MRS and MSR instructions move the contents of the Application Program Status Register (APSR) to or from an
ARM core register, see:
• MRS on page A8-496
• MSR (immediate) on page A8-498
• MSR (register) on page A8-500.

At system level, software can also:
• use these instructions to access the SPSR of the current mode
• use the CPS instruction to change the CPSR.M field and the CPSR.{A, I, F} interrupt mask bits.

#### A4.6 Load/store instructions
Fixme [Table A4-12 Load/store instructions] Page175

#### A4.7 Load/store multiple instructions
Load Multiple instructions load a subset, or possibly all, of the ARM core registers from memory.
Store Multiple instructions store a subset, or possibly all, of the ARM core registers to memory.

Fixme [Table A4-13 Load/store multiple instructions] Page177

#### A4.8 Miscellaneous instructions
Fixme [Table A4-14 Miscellaneous instructions] Page178

#### A4.9 Exception-generating and exception-handling instructions
The following instructions are intended specifically to cause a synchronous processor exception to occur:
• The SVC instruction generates a Supervisor Call exception. For more information, see Supervisor Call (SVC)
exception on page B1-1210.
• The Breakpoint instruction BKPT provides software breakpoints. For more information, see About debug
events on page C3-2038.
• In a processor that implements the Security Extensions, when executing at PL1 or higher, the SMC instruction
generates a Secure Monitor Call exception. For more information, see Secure Monitor Call (SMC) exception
on page B1-1211.
• In a processor that implements the Virtualization Extensions, in software executing in a Non-secure PL1
mode, the HVC instruction generates a Hypervisor Call exception. For more information, see Hypervisor Call
(HVC) exception on page B1-1212.

Fixme [Table A4-15 Exception-generating and exception-handling instructions] Page179

#### A4.10 Coprocessor instructions
There are three types of instruction for communicating with coprocessors. These permit the processor to:
• Initiate a coprocessor data-processing operation. For details see CDP, CDP2 on page A8-358.
• Transfer ARM core registers to and from coprocessor registers. For details, see:
	— MCR, MCR2 on page A8-476
	— MCRR, MCRR2 on page A8-478
	— MRC, MRC2 on page A8-492
	— MRRC, MRRC2 on page A8-494.
• Load or store the values of coprocessor registers. For details, see:
	— LDC, LDC2 (immediate) on page A8-392
	— LDC, LDC2 (literal) on page A8-394
	— STC, STC2 on page A8-662.

#### A4.11 Advanced SIMD and Floating-point load/store instructions
Fixme [Table A4-16 Extension register load/store instructions] Page181
Fixme [Table A4-17 Element and structure load/store instructions] Page181

#### A4.12 Advanced SIMD and Floating-point register transfer instructions
Fixme [Table A4-18 Extension register transfer instructions] Page183

#### A4.13 Advanced SIMD data-processing instructions
Advanced SIMD data-processing instructions process registers containing vectors of elements of the same type
packed together, enabling the same operation to be performed on multiple items in parallel.

Fixme [Figure A4-2 Advanced SIMD instruction operating on 64-bit registers] Page184

##### A4.13.1 Advanced SIMD parallel addition and subtraction

Fixme [Table A4-19 Advanced SIMD parallel add and subtract instructions] Page185

##### A4.13.2 Bitwise Advanced SIMD data-processing instructions
Fixme [Table A4-20 Bitwise Advanced SIMD data-processing instructions] Page186

##### A4.13.3 Advanced SIMD comparison instructions
Fixme [Table A4-21 Advanced SIMD comparison instructions] Page186

##### A4.13.4 Advanced SIMD shift instructions
Fixme [Table A4-22 Advanced SIMD shift instructions] Page187

##### A4.13.5 Advanced SIMD multiply instructions
Fixme [Table A4-23 Advanced SIMD multiply instructions] Page188

##### A4.13.6 Miscellaneous Advanced SIMD data-processing instructions
Fixme [Table A4-24 Miscellaneous Advanced SIMD data-processing instructions] Page189

#### A4.14 Floating-point data-processing instruction
Fixme [Table A4-25 Floating-point data-processing instructions] Page191


### A5. ARM Instruction Set Encoding
#### A5.1 ARM instruction set encoding
__The ARM instruction stream is a sequence of word-aligned words. Each ARM instruction is a single 32-bit word__ in
that stream. The encoding of an ARM instruction is:

Fixme [32 bits instruction structure] page194

Fixme [Table A5-1 ARM instruction encoding] page194

##### A5.1.1 The condition code field
This field contains one of the values 0b0000-0b1110, as shown in Table A8-1 on page A8-288.
Fixme [Table A8-1 Condition codes]page288

#### A5.2 Data-processing and miscellaneous instructions
Fixme [data process instructions structure]page196

Fixme [Table A5-2 Data-processing and miscellaneous instructions]page196

其余指令可参照此命令，只是op 的不同。

### A6. Thumb Instruction Set Encoding
#### A6.1 Thumb instruction set encoding
The Thumb instruction stream is a sequence of halfword-aligned halfwords. Each Thumb instruction is either a
single 16-bit halfword in that stream, or a 32-bit instruction consisting of two consecutive halfwords in that stream.
If the value of bits[15:11] of the halfword being decoded is one of the following, the halfword is the first halfword
of a 32-bit instruction:
• 0b11101
• 0b11110
• 0b11111.
Otherwise, the halfword is a 16-bit instruction

疑问点：thumb 是16bit 或32bit对齐，那怎么与32 bit的ARM 指令集怎么区分？
当ARM 处于ARM state，CPU 将会按照32 bit ARM指令集去取指并执行，反之使用16、32bit 的thumb 指令集解析。

#### A6.2 16-bit Thumb instruction encoding
Fixme [16-bit thumb instruction encoding]page223

Fixme [Table A6-1 16-bit Thumb instruction encoding]page223

#### A6.3 32-bit Thumb instruction encoding
Fixme [ 32-bit Thumb instruction encoding]page230

Fixme [Table A6-9 32-bit Thumb instruction encoding]page230

### A7 Advanced SIMD and Floating-point Instruction Encoding
skip

### A9 The ThumbEE Instruction Set
#### A9.1 About the ThumbEE instruction set
In general, __instructions in ThumbEE are identical to Thumb instructions__, with the following exceptions:
• A small number of instructions are affected by modifications to transitions from ThumbEE state. For more
information, see ThumbEE state transitions.

• __A substantial number of instructions have a null check on the base register before any other operation takes__
place, but are identical (or almost identical) in all other respects. For more information, see Null checking on
page A9-1113.

• A small number of instructions are modified in additional ways. See Instructions with modifications on
page A9-1113.

• Three Thumb instructions, BLX (immediate), 16-bit LDM, and 16-bit STM, are removed in ThumbEE state.
The encoding corresponding to BLX (immediate) in Thumb is UNDEFINED in ThumbEE state.
16-bit LDM and STM are replaced by new instructions, for details see Additional ThumbEE instructions on
page A9-1123.

• Two new 32-bit instructions, ENTERX and LEAVEX, are introduced in both the Thumb instruction set and the
ThumbEE instruction set. See Additional instructions in Thumb and ThumbEE instruction sets on
page A9-1116. These instructions use previously UNDEFINED encodings.

__Attempting to execute ThumbEE instructions at PL2 is UNPREDICTABLE.__

##### A9.1.1 ThumbEE state transitions
Instruction set state transitions to ThumbEE state can occur implicitly as part of a return from exception, or
explicitly on execution of an __ENTERX instruction.__

Instruction set state transitions from ThumbEE state can only occur due to an exception, or due to a transition to
Thumb state using the __LEAVEX instruction.__ Return from exception instructions (RFE and SUBS PC, LR, #imm) are
UNPREDICTABLE in ThumbEE state.

##### A9.1.2 Null checking
A null check is performed for all load/store instructions when they are executed in ThumbEE state. If the value in
the base register is zero, execution branches to the NullCheck handler at HandlerBase – 4.

#### A9.2 ThumbEE instruction set encoding
In general, instructions in the ThumbEE instruction set are encoded in exactly the same way as Thumb instructions
described in Chapter A6 Thumb Instruction Set Encoding. The differences are as follows:
• There are no 16-bit LDM or STM instructions in the ThumbEE instruction set.
• The 16-bit encodings used for LDM and STM in the Thumb instruction set are used for different 16-bit
instructions in the ThumbEE instruction set. For details, see 16-bit ThumbEE instructions.
• There are two new 32-bit instructions in both Thumb state and ThumbEE state. For details, see Additional
instructions in Thumb and ThumbEE instruction sets on page A9-1116.

##### A9.2.1 16-bit ThumbEE instructions
Fixme [Table A9-2 16-bit ThumbEE instructions]page1115

#### A9.3 Additional instructions in Thumb and ThumbEE instruction sets
On a processor with the ThumbEE Extension, there are two additional 32-bit instructions, ENTERX and LEAVEX. These
are available in both Thumb state and ThumbEE state.
##### A9.3.1 ENTERX, LEAVEX
ENTERX causes a change from Thumb state to ThumbEE state, or has no effect in ThumbEE state.
ENTERX is UNDEFINED in Hyp mode.
LEAVEX causes a change from ThumbEE state to Thumb state, or has no effect in Thumb state.

### A8 Instruction Descriptions

#### A8.2 Standard assembler syntax fields
The following assembler syntax fields are standard across all or most instructions:
<c> Is an optional field. It specifies the condition under which the instruction is executed. See
	Conditional execution on page A8-288 for the range of available conditions and their encoding. If
	<c> is omitted, it defaults to always (AL).

<q> Specifies optional assembler qualifiers on the instruction. The following qualifiers are defined:
	.N Meaning narrow, specifies that the assembler must select a 16-bit encoding for the
	instruction. If this is not possible, an assembler error is produced.
	.W Meaning wide, specifies that the assembler must select a 32-bit encoding for the
	instruction. If this is not possible, an assembler error is produced.

#### A8.3 Conditional execution	
Most ARM instructions, and most Thumb instructions from ARMv6T2 onwards, can be executed conditionally,
based on the values of the APSR condition flags.

Fixme [Table A8-1 Condition codes]page288
