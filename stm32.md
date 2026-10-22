</p>

This memory specification outlines a typical microcontroller or embedded system memory layout where 128 KB of System SRAM is divided into two distinct physical or logical blocks: a 112 KB region and a 16 KB region.
Together, these regions must accommodate all runtime data: the stack, heap, and static/global variables.

Memory Breakdown & Allocation Strategy

Because the SRAM is split, developers typically allocate the memory sections based on performance needs, data sizes, and hardware architecture:
• Static/Global Variables (.data and .bss segments):
	• These have a fixed size known at compile time.
	• They are usually placed in the larger 112 KB block to ensure there is enough room for large arrays, peripheral buffers, and system states.
• The Stack:
	• Used for function call tracking, local variables, and interrupt contexts.
	• Because the stack requires fast, predictable access and has a strict last-in, first-out (LIFO) structure, it is often placed entirely within the 16 KB block (if 16 KB is sufficient for the call depth) or allocated a dedicated chunk of the 112 KB block.
• The Heap:
	• Used for dynamic memory allocation (malloc, free, new).
	• The heap is highly volatile and prone to fragmentation. It usually occupies the remaining unallocated space in the 112 KB block, growing toward the static variables.

Common Architectural Reasons for the Split

The 112 KB + 16 KB configuration is highly characteristic of specific hardware architectures, such as certain ARM Cortex-M microcontrollers (e.g., STM32 series), and is usually done for two reasons:
1. Tightly-Coupled Memory (TCM / CCM): The 16 KB block might be Core Coupled Memory (CCM) or Instruction/Data Tightly-Coupled Memory (ITCM/DTCM). This memory is wired directly to the processor's core via a dedicated bus, allowing for zero-wait-state execution. Developers often put the Stack or critical interrupt service routines (ISRs) here for maximum performance.
2. Multi-bus Matrix (Concurrency): By splitting the SRAM into two blocks on separate buses, the CPU can read/write to the 16 KB block (e.g., handling stack operations) at the exact same time a DMA (Direct Memory Access) controller is moving data into the 112 KB block from a peripheral (like a UART or SPI bus) without stalling the processor.

</p>

• Peripheral Addresses: To turn on an LED, configure a timer, or send data over UART, you must look at the chip's Memory Map to find the exact 32-bit base address of that peripheral's registers.
• Linker Scripts: When compiling a bare-metal program, you must write or modify a Linker Script (.ld file). This file tells the compiler exactly where the executable code (.text) goes in Flash memory and where variables (.data, .bss) go in SRAM based on the memory map.




<p>


Example Memory Map & Topology (STM32F4 Reference)

In microcontrollers like the STM32F4 Series Memory and Bus Architecture, a 32-bit multi-layer AHB bus matrix connects:

AHB Masters (Initiators)	AHB Slaves (Targets / Memory Map Regions)
• Cortex-M4 I-Bus (Instruction)
• Cortex-M4 D-Bus (Data)
• Cortex-M4 S-Bus (System)
• DMA1 Memory Bus
• DMA2 Memory / Peripheral Bus
• Ethernet DMA Bus
• USB OTG HS DMA Bus	• Internal Flash Memory (ICode / DCode)
• Main Internal SRAM1 (112 KB)
• Auxiliary Internal SRAM2 (16 KB)
• SRAM3 (64 KB on select devices)
• AHB1 Peripherals & APB Bridges
• AHB2 Peripherals
• FSMC (Flexible Static Memory Controller)


Key Memory Map Characteristics

• Flat Address Space: All AHB slaves map into a single unified 32-bit (4 GB) flat address space defined by the SoC vendor.
• Simultaneous Access Example: The CPU can fetch instructions from Flash via the I-Bus while DMA2 simultaneously writes incoming network packets into SRAM1 over a separate parallel layer with zero contention.
• Non-Matrix Memories: Specialized tightly coupled memories—such as a Cortex-M4 Core Coupled Memory (CCM RAM)—often bypass the bus matrix entirely to provide single-cycle, zero-wait-state local access exclusively to the CPU core

</p>



**2. The Bus Matrix Prevents Performance Bottlenecks

A multi-AHB bus matrix allows parallel data transfers. If you don't understand how your specific chip's matrix is routed, your bare-metal application might run incredibly slow or stall.
• Concurrency: If you configure a DMA controller to move data from a UART peripheral into the exact same SRAM block where the CPU is currently doing heavy math, they will fight for the bus. This causes bus contention, inserting wait states that slow down your CPU.
• Optimization: By looking at the bus matrix diagram, you can optimize your code. For example, you can tell the DMA to write to SRAM2 while the CPU uses SRAM1. Because they are on different slave ports of the matrix, both happen simultaneously with zero slowdown.**


