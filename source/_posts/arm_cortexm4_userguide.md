

arm_cortexm4_processor_trm_100166_0001_00_en

## 1. 基础
### 1.1. 特性

FixMe
![cortex-m4 block diagram]()

arm cortex-m4 特性：
- Nested Vectored Interrupt Controller(NVIC)与processor core 集中很近以提供低延迟中断处理
    - 外部中断1-240
    - 优先级设定位 3-8bit
    - 支持动态设定优先级
    - 支持优先级组，以此来支持preempt
- low-power (A low gate count 门栅极数量少), low interrupt latency
    - Banked stack pointer(SP)
    - a subset of the Thumb instruction set, defined in ARMv7-m ARM(architechture reference manual)
    - 硬件支持整数SDIV, UDIV
    - 支持可中断继续的 LDM, STM, PUSH, POP for low interrupt latency
    - 自动保存/恢复处理器状态for low interrupt latency entry/exit

- optional floating point unit(FPU)
    - 32bit 单精度指令
    - 三条独立处理流水线(stage pipeline)
- optional memory protection unit(MPU)
    - eight memory regions

- bus interface
    - 支持三种AHB （Advanced High-performance Bus-lite）接口: Icode, Dcode, System Bus interface
    - private peripheral Bus(PPB) 实现基于APB(Advanced Peripheral Bus) interface
    - 支持atomic bit-band 读写操作，bit-band 对bit 进行操作，而非需要对齐的R/W
    - Mem access align
    - wirte buffer for write data
    - Exclusive access transfer for multiprocessor systems
  
- low-cost debug
    - breakpoints and code patches
    - watchpoints, tracing, system profiling
    - bridge to a trace port analyzer


### 1.2. Configurable options
You can configure your Cortex-M4 implementation to include optional components, such as a Memory
Protection Unit (MPU), a Flash Patch and Breakpoint Unit (FPB), and a Data Watchpoint and Trace Unit
(DWT).
The following optional components can be configured for the Cortex-M4 processor:
- Memory Protection Unit (MPU) .
- Flash Patch and Breakpoint Unit (FPB).
- Data Watchpoint and Trace Unit (DWT).
- Instrumentation Trace Macrocell Unit (ITM).
- Embedded Trace Macrocell (ETM). See theETM-M4 Technical Reference Manual.
- Advanced High-performance Bus Access Port (AHB-AP).
- AHB Trace Macrocell (HTM) interface.
- Trace Port Interface Unit (TPIU).
- Wake-up Interrupt Controller (WIC).
- Debug Port AHB-AP interface.
- Floating-Point Unit (FPU).
- Bit-banding.
- Constant AHB control


### 1.3. 相关协议
`ARM architecture`
The processor implements the ARMv7E-M architecture profile.
See the ARM®v7-M Architecture Reference Manual.

`Bus architecture`
The processor implements an interface for CoreSight and other debug components using the AMBA 3
APB protocol.
The processor provides three primary bus interfaces implementing a variant of the AMBA 3 AHB-Lite
protocol. See:
- The ARM® AMBA® 3 AHB-Lite Protocol (v1.0).
- The ARM® AMBA® 3 APB Protocol Specification.

`Debug`
The debug features of the processor implement the ARM debug interface architecture.
See the ARM® Debug Interface v5 Architecture Specification.

`Embedded Trace Macrocell`
The trace features of the processor implement the ARM Embedded Trace Macrocell architecture.
See the ARM® Embedded Trace Macrocell Architecture Specification.

`Floating Point Unit`
The Cortex-M4 FPU implements ARMv7E-M architecture with FPv4-SP extensions


## 2. Programmers model
**Mode, privilege and stack pointer are key concepts used in ARMv7-M.**

### 2.1. Mode, Privilege and Stack pointer
#### 2.1.1. Mode 

`Thread mode`
Is entered on reset, and can be entered as a result of an exception return.

`Handler mode`
Is entered as a result of an exception. The processor must be in Handler mode to issue an exception return.

#### 2.1.2. Privilege 
Code can execute as privileged or unprivileged. Unprivileged execution limits or excludes access to
some resources. Privileged execution has access to all resources. **Execution in Handler mode is always privileged.** Execution in Thread mode can be privileged or unprivileged.

#### 2.1.3. Stack pointer
The processor implements a banked pair of stack pointers: 
- the Main stack pointer  
- the Process stack pointer  

**In Handler mode, the processor uses the Main stack pointer.**
**In Thread mode it can use either stack pointer.**

`Permission`
- Handler mode is always privileged. 
- Thread mode can be privileged or unprivileged

FixMe
![mode privilege stack pointer relationship]()

### 2.2. memory model
bus matrix(总线矩阵)会去仲裁访问MEM, system ctrl, debug, support unaligned access的优先级。

Armv7-m system address map 如下：

FixMe
![system address map]()

FixMe
![memory region]()

private peripheral bus
The internal Private Peripheral Bus (PPB) interface provides access to:
- The Instrumentation Trace Macrocell (ITM).
- The Data Watchpoint and Trace (DWT).
- The Flashpatch and Breakpoint (FPB).
- The System Control Space (SCS), including the Memory Protection Unit (MPU) and the Nested
Vectored Interrupt Controller (NVIC).

The external PPB interface provides access to:
- The Trace Point Interface Unit (TPIU).
- The Embedded Trace Macrocell (ETM).   
- The ROM table.
- Implementation-specific areas of the PPB memory map.

`unaligned access`
They are converted into two or more aligned accesses by the DCode and System bus
interfaces.

unaligned access
support | unsupport
:- | :-
singles load/store | load/store double support word aligned access, others generate a fault
cross memory map boundaries are UNPREDICATABLE | 
bit-band region are UNPREDICATABLE

Note:
All Cortex-M4 external accesses are aligned.

`Exclusive monitor`
支持几乎所有mem， cortex-m4 不支持bit-band regions exclusive access。

`bit-banding`
>bit-banding map a complete word memory into a single bit in the bit-band region. 

一个word (32bit) 映射成一个bit。
有两个对应的32M alias regions 映射 1M bit-band region:
- SRAM 
即bit-region 映射一个word mem 空间成独立的bit 。

### 2.3. Register
FixMe
![cortex-m4 registers]()

- 13个通用寄存器 R0 ~ R12. R0-R7 可以被所有指令访问，R8-R12 支持32-bit指令访问，多数16-bit指令不支持访问
- SP, LR, PC. Handle mode 只可以使用SP_main， Thread mode 可以使用SP_process, SP_main
- program status registers, xPSR 状态寄存器

`Program Status Register`
three subregisters:
- Application Program Status Register, APSR
- Interrupt Program Status Register, IPSR
- Execution Program Status Register, EPSR

### 2.4. Exception
exception 工作在handle mode, CPU 自动保存/恢复 state.

The ARMv7-M profile supports the following exceptions:
- Reset
- NMI  (Non-Maskable Interrupt) is the highest priority exception other than reset. It is permanently enabled with a fixed priority of -2.
- HardFault
- MemManage  handles memory protection faults that are determined by the Memory Protection Unit or by fixed memory protection constraints
- BusFault  handles memory-related faults, other than those handled by the MemManage fault
- UsageFault
  - Undefined Instruction.
  - Invalid state on instruction execution.
  - Error on exception return.
  - Attempting to access a disabled or unavailable coprocessor.
- Supervisor Call (SVCall)
- Interrupt

FixMe
![excepetion number]()

#### 2.4.1. states
state | remarks
:-: | :-
Inactive | not pending or active
Pending | An exception that has been generated, but that the processor has not yet started processing.
Active | An exception for which the processor has started executing a corresponding exception handler, but has not returned from that handler. The handler for an active exception is either running or preempted by the handler for a higher priority exception.
Active and pending | One instance of the exception is active, and a second instance of the exception is pending. 

#### 2.4.2. vector table
The vector table contains the initialization value for the stack pointer, and the entry point addresses of each
exception handler. 

Fixme
![vector_table]() 
Software can find the current location of the table, or relocate the table, using the VTOR(Vector Table Offset Register)

#### 2.4.3. Priority
There are execution priority, exception priority. An exception whose exception priority is sufficiently higher than the execution priority becomes active. 

- a lower priority value indicating a higher priority
- configurable priorities for all exceptions except Reset, HardFault, and NMI.

To increase priority control in systems with interrupts, the NVIC supports priority grouping.
This divides each interrupt priority register entry into two fields:
- an upper field that defines the group priority
- a lower field that defines a subpriority within the group.

#### 2.4.4. Exception return
- If the exception state is active and pending:
    - If the exception has sufficient priority, it becomes active and the processor reenters the exception handler.
    - Otherwise, it becomes pending.
- If the exception state is active it becomes inactive.
- The processor restores the information that it stacked on exception entry.
- If the code that was preempted by the exception handler was running in Thread mode the processor changes to Thread mode.

### 2.5. Fault handling 
#### 2.5.1. types
Faults are generated by:
- a bus error on:
    - an instruction fetch or vector table load
    - a data access.
- an internally-detected error such as an undefined instruction
- attempting to execute an instruction from a memory region marked as Execute-never
(XN).
- If your device contains an MPU, a privilege violation or an attempt to access an
unmanaged region causing an MPU fault

FixMe
![fault_types_1]()
![fault_types_2]()

#### 2.5.2. fault status register and address register
The fault status registers indicate the cause of a fault. The fault address register indicates the address accessed by the operation that caused the fault.

FixMe
![fault_sts_and_addr_register]()

## 3. coretex-m4 peripherals
FixMe
![Core_peripheral_register_regions]()

### 3.1. NVIC(Nested Vectored Interrupt Controller)
The NVIC supports: 
- up to 240 interrupts
- 256 levels of priority, changed dynamically
- Level and pulse detection of interrupt signals
- Grouping of priority values into group priority and subpriority fields
- An external Non Maskable Interrupt (NMI)
- NVIC registers are located within the SCS.
- All NVIC registers and system debug registers are little-endian regardless of the endianness state of the processor

The processor and NVIC can be put into a very low-power sleep mode, leaving the Wake Up Controller (WIC) to identify and prioritize interrupts. The processor fully implements the Wait For Interrupt (WFI), Wait For Event (WFE) and the Send Event (SEV) instructions. 

You can only fully access the NVIC from privileged mode. Any other user mode access causes a bus fault.


FixMe
![nvic registers]()

### 3.2. System Control
FixMe
![system_control_registers]()

### 3.3. system timer
The processor has a **24-bit system timer**, SysTick, that **counts down from the reload value to zero**, reloads, that is wraps to, the value in the SYST_RVR register on the next clock edge, then
counts down on subsequent clocks.

FixMe
![system_timer_reg]()

### 3.4. MPU(memory protection unit)

The MPU divides the memory map into a number of regions, and defines the location, size,
access permissions, and memory attributes of each region. It supports:
- independent attribute settings for each region
- overlapping regions
- export of memory attributes to the system.

FixMe
![mem_attr]()

FixMe
![mpu registers]()


## Reference
Cortex™-M4 Devices Generic User Guide(DUI0553.pdf)
