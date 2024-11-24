# C PlayStation Demo

A demo for the original PlayStation, created using C and MIPS assembly alongside the PsyQ x32 library provided by Psygnosis and Sony for the PlayStation SDK.
![tv-psygnosis-transparent](https://github.com/user-attachments/assets/f6f99b48-7f5e-472f-859b-5e51af403cf0)

## Table of Contents
1. [Project Overview](#project-overview)
2. [Hardware Overview](#playstation-1-hardware-overview)
   - [CPU](#more-on-the-cpu)
   - [Memory](#more-on-memory-and-endianness)
   - [GPU](#more-on-the-gpu)
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
<sup>A simple overview of key components on the PS1's motherboard.</sup>

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

CP1, the FPU (Floating Point Unit), was a coprocessor typically dedicated to handling floating point numbers. The lack of this coprocessing unit in the PS1 partly results in snapping vertices and affine texture mapping, producing the "warbling" of geometry which has become synonymous with the PS1 (we'll further elaborate on this in [*More on the GPU*](#more-on-the-gpu)). The version of the MIPS R3000A that made its way into Sony's first home console was a lower-end processor, which made it more affordable and a practical choice for their first foray into gaming hardware.

### More on Memory and Endianness
MIPS memory consists of 32-bit memory addresses, 2<sup>32</sup> addressable bytes. The PlayStation does not use an MMU (Memory Management Unit), so it has no virtual memory. It's entirely physical.

The PlayStation is *little endian*. If we wanted to store the hexadecimal value 0x12345678 in memory, the bytes would be populated, from left to right: [7 8][5 6][3 4][1 2]. In a *big endian* machine, it would populate, left to right: [1 2][3 4][5 6][7 8]. MIPS CPUs typically allow a choice between little and big endianness, but the PlayStation is hardwired this way.

#### On the CPU Memory Map
The following section is a breakdown of the PS1's CPU memory map.

![ps1-cpu-memory-map](https://github.com/user-attachments/assets/a929df05-9dd3-455b-86f9-7680de79e85b)
<sup>A map of the PS1's CPU memory addresses.</sup>

It's worth briefly noting that, in standard MIPS processors, the KUSEG would contain 2GB of virtual memory. Since the PlayStation doesn't support this, its first 512MB instead serve as a mirror of KSEG0 and KSEG1.

Some examples of I/O ports at addresses starting with 0x1F80:

- GPU (video) Registers:
  - 0x1F801810 write = GP0, where packets of information are sent, such as triangles to draw, color information, shading information, etc.
  - 0x1F801814 write = GP1, where packets of information are sent to configure display control, such as bit depth, resolution, NTSC/PAL, etc.
  - 0x1F801810 read = Read responses to GP0 and GP1 commands.
  - 0x1F801814 read = Read GPU status register.

- SPU (sound) Control Registers:
  - 0x1F801D80 = Main volume control, left and right.
  - 0x1F801D84 = Reverb output volume control, left and right.
  - 0x1F801D88 = Voice 0..23 Key ON (Start Attack, Decay, or Sustain).
  - 0x1F801D8C = Voice 0..23 Key OFF (Start release).

- MDEC (motion) Registers:
  - 0x1F801820 write = MDEC Command and Parameter Register
  - 0x1F801824 write = MDEC Control and Reset Register
  - 0x1F801820 read = MDEC Data and Response Register
  - 0x1F801824 read = MDEC Status Register

### More on the GPU
Did you know that the PlayStation's GPU is a 2D rasterizer? Yes, it is only capable of drawing 2D objects. In this section, we will elaborate on the PlayStation's iconic visual quirks, as referenced briefly in the section [*More on the CPU*](#more-on-the-cpu).

#### Types of Primitives
First, there are a few primitives that the PlayStation is capable of drawing:

![primitives-in-action](https://github.com/user-attachments/assets/7bd9b31f-1cfc-4711-a774-b4ac3b7c02ea)
<sup>An example of a sprite (Gex from *Gex*), gouraud shading (Chocobo from *FFVII*), and textured polygons (Harry Mason from *Silent Hill*).</sup>

1. **Flat-Shaded Polygons** - A triangle or quad painted with a single color.
2. **Gouraud-Shaded Polygons** - Where tris or quads are painted with different colors per vertex then interpolated per pixel to create the facade of a smooth-shaded model without needing textures.
3. **Textured Polygons** - Where a texture image is loaded into VRAM before being applied to a polygon using UV coordinates.
4. **Lines** - Lines drawn between two screen coordinates.
5. **Sprites** - Essentially texture images with scale and location coordinates, akin to the demons of *Doom*.

#### Drawing Primitives
As you can see in the [CPU Memory Map diagram](#on-the-cpu-memory-map), the PlayStation's VRAM is not memory mapped. Primitives must be drawn to the frame buffer by asking the GPU to do so. We can write to GP0, as referenced in the same section, telling it which type of primitive we want it to draw, along with the necessary parameters (vertex locations and color). This information is quickly sent, via direct memory access, to the GPU in a sequence of packets containing 32-bit values.

At this stage of the rasterization process, all polygon coordinates being loaded and stored in the memory mapped GPU I/O port are in 2D. This lack of access to the z-axis at the rasterization stage will play into one of the PlayStation's more prominent visual artifacts, which will be covered below alongside other artifacts synonymous with Sony's first home console.

#### Texture Wobble

![texture-mapping](https://github.com/user-attachments/assets/1a194c29-da77-48a4-a26c-ff45766728c6)
<sup>Going from an untextured mesh to a textured mesh using inverse texture mapping.</sup>

![affine-perspective](https://github.com/user-attachments/assets/e301568f-a8b2-4010-af3a-d2375bb9791d)
<sup>This is a side-by-side comparison of affine texture mapping and perspective correct texture mapping, respectively, as visualized by my [simple rasterization software](https://github.com/Nico-Posateri/c-software-rasterizer).</sup>

#### Polygon Pop

#### Polygon Jitter

#### Circling Back to The FPU

## Project Resources
The following section contains an overview of the languages, emulators, and libraries used to complete this project.

### MIPS Assembly - Instructions and Syntax
<!-- COMMENTED OUT UNTIL FINISHED . . .
### PsyQ SDK
![psyq](https://github.com/Nico-Posateri/c-playstation-demo/assets/141705409/be5f2348-d887-42e2-ba3e-0b204459d29e)
### Emulation
-->

## Additional Information
Plenty of technical information on the PlayStation's hardware featured in this document was gleaned from varied sources, but a bulk of it comes from Gustavo Pezzi's [PS1 Programming with MIPS Assembly and C course](https://pikuma.com/courses/ps1-programming-mips-assembly-language), as well as *PlayStation Architecture: A Promising Newcomer*, a practical analysis of the PlayStation written by Rodrigo Copetti.

Moreover, this [rough breakdown on CISC and RISC architecture published by Stanford students](https://cs.stanford.edu/people/eroberts/courses/soco/projects/risc/risccisc/) in 2000 was an interesting read.
