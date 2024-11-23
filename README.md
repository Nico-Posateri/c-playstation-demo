# C PlayStation Demo

A demo for the original PlayStation, created using C and MIPS assembly alongside the PsyQ x32 library provided by Psygnosis and Sony for the PlayStation SDK.
![tv-psygnosis-transparent](https://github.com/user-attachments/assets/f6f99b48-7f5e-472f-859b-5e51af403cf0)

# Table of Contents
1. [Project Overview](#project-overview)
2. [Hardware Overview](#playstation-1-hardware-overview)
   - [CPU](#more-on-the-cpu)
   - [Memory](#more-on-memory-and-endianness)
   - [Memory Map](#on-the-cpu-memory-map)
4. [Project Resources](#project-resources)
   - [MIPS Assembly](#mips-assembly---instructions-and-syntax)
   - [PsyQ SDK](#psyq-sdk)
   - [Emulation](#emulation)
5. [Additional Information](#additional-information)

## Project Overview
Before I began this project, I simply wanted to learn to develop a demo using the original PS1 SDK and the C language. Shortly after starting it, however, I thought this would be more fun if I embraced it as a broader learning experience.

This is why, as you might have seen from the table of contents, this readme serves as an interesting, albeit cursory, breakdown of the first commercial PS1's hardware, MIPS Assembly, the PsyQ SDK, and emulation.

## PlayStation 1 Hardware Overview
The PlayStation 1 model SCPH-1000 was released in Japan on December 3, 1994. This was the first commercial unit, featuring an S-video port, parallel I/O ports, and RCA connectors, all of which would be removed in subsequent revisions.

![Sony-PlayStation-SCPH-1000-Motherboard-Top](https://github.com/user-attachments/assets/8cb815a8-15d8-4397-beae-d0bf6497feb8)

- **CPU**: 32-bit R3000A MIPS, 33.86 MHz (manufactured by LSI)
  - 32 general-purpose registers
  - 2 dedicated multiplication and division registers
  - 32-bit data bus
  - 32-bit address bus
  - 1 ALU (Arithmetic Logic Unit for adding, subtracting, and logical operators)
  - 5-stage pipeline (can parallelize instructions)
- **Coprocessors**:
  - **CP0**: System Control
  - **CP2**: GTE (Geometry Transformation Engine)
  - **MDEC** (Motion Decoder)
- **RAM**: 2 MB EDO Memory (4 x 512 KB chips)
- **VRAM**: 1 MB (2 x 512 KB chips)
- **DRAM**: 512 KB (dedicated to sound)
- **GPU**: SCPH-9000 2D rasterizer
- **SPU**: Sound Processing Unit, 16-bit 24-channel ADPCM (Adaptive Differential Pulse-Code Modulation)
- **CD Subsystem**: Motor and laser control

### More on the CPU
The 1990s saw a rise in the commonality of RISC (Reduced Instruction Set Computer) over CISC (Complex Instruction Set Computer) processing. The PS1 features a RISC processor. RISC philosophy aims to execute simple instructions that can be completed in a single clock cycle.

The PlayStation's MIPS (Microprocessor without Interlocked Pipelined Stages) CPU was a unique R3000A processor from MIPS and LSI Logic. MIPS created the instruction set for the MIPS RISC architecture. They then licensed out the IP so that manufacturers could produce custom MIPS chips. LSI Logic was one such chip manufacturer, which would produce custom chips for businesses like Sony, who commissioned the manufacturing of the LSI CW33300 from LSI Logic. This chip was binary-compatible with the MIPS R3000A.

CP1, the FPU (Floating Point Unit), was a coprocessor typically dedicated to handling floating point numbers. The lack of this coprocessing unit in the PS1 results in snapping vertices and affine texture mapping, producing the "warbling" of geometry which has become synonymous with the PS1. The version of the MIPS R3000A that made its way into Sony's first home console was a lower-end processor, which made it more affordable and a practical choice for their first foray into gaming hardware.

### More on Memory and Endianness
MIPS memory consists of 32-bit memory addresses, 2<sup>32</sup> addressable bytes. The PlayStation does not use an MMU (Memory Management Unit), so it has no virtual memory. It's entirely physical.

The PlayStation is *little endian*. If we wanted to store the hexadecimal value 0x12345678 in memory, the bytes would be populated, from left to right: [7 8][5 6][3 4][1 2]. In a *big endian* machine, it would populate, left to right: [1 2][3 4][5 6][7 8]. MIPS CPUs typically allow a choice between little and big endianness, but the PlayStation is hardwired this way.

### On the CPU Memory Map
The following section is a breakdown of the PS1's CPU memory map.

![ps1-cpu-memory-map](https://github.com/user-attachments/assets/a929df05-9dd3-455b-86f9-7680de79e85b)

## Project Resources
The following section contains an overview of the languages, emulators, and libraries used to complete this project.
<!-- COMMENTED OUT UNTIL FINISHED . . .
### MIPS Assembly - Instructions and Syntax
### PsyQ SDK
![psyq](https://github.com/Nico-Posateri/c-playstation-demo/assets/141705409/be5f2348-d887-42e2-ba3e-0b204459d29e)
### Emulation
-->
## Additional Information
[PS1 Programming with MIPS Assembly and C course](https://pikuma.com/courses/ps1-programming-mips-assembly-language) taught by [Gustavo Pezzi](https://github.com/gustavopezzi).
<!-- COMMENTED OUT UNTIL FINISHED . . .
https://cs.stanford.edu/people/eroberts/courses/soco/projects/risc/risccisc/
https://www.copetti.org/writings/consoles/playstation/
-->
