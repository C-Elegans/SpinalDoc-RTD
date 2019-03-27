
VexRiscv (RV32IM CPU)
=====================

VexRiscv is a fpga friendly RISC-V ISA CPU implementation with the following features:


* RV32IM instruction set
* 5 stage pipeline (Fetch, Decode, Execute, Memory, WriteBack)
* 1.44 DMIPS/Mhz when all features are enabled
* Optimized for FPGA
* Optional MUL/DIV extension
* Optional instruction and data caches
* Optional MMU
* Optional debug extension allowing eclipse debugging via a GDB >> openOCD >> JTAG connection
* Optional interrupts and exception handling with the Machine and the User mode from the riscv-privileged-v1.9.1 spec.
* Two implementations of shift instructions, Single cycle or ``shiftNumber`` cycles
* Each stage can have bypass or interlock hazard logic
* FreeRTOS port https://github.com/Dolu1990/FreeRTOS-RISCV

Much more information is available here: `https://github.com/SpinalHDL/VexRiscv <https://github.com/SpinalHDL/VexRiscv>`_
