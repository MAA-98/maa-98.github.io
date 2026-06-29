---
layout: post
title: "Unix: Process Memory Model"
date: 2026-06-29
math: true
---

# Unix: Process Memory Model

The simple von Neumann architecture stores program instructions and data in the same memory unit. The Unix process uses this as a model of the available memory, providing one long contiguous address space, which C programs then make use of with pointers.

## Basics

Preliminaries:
- A *bit* is a 0 or 1 binary value,
- A *byte* is 8 bits in sequence, having 2^8 = 256 possible distinct values.
- Each byte has an address. 32-bit architectures have 32 bits or 4 bytes sized addresses, 64-bit architectures have 64 bits for address space, or 8 bytes. Note that, taken literally, this would require a contiguous memory of 2^64 bytes ~ 16,384 petabytes which is practically unfeasible, so the memory addresses are *virtual*: each address does not have an actual space in the hardware, but is used by the CPU to map to physical memory using page tables managed by the OS.
- In 64-bit systems, a *word* often refers to a sequence of 8 bytes, so an address is word-sized.
- Addresses are displayed using base-16, with 0-F digits. For 64-bit addresses, you need two base-16 digits to distinguish bytes, so 16 for all 64-bit addresses.
- Integers in base 16 are displayed with a prefix `0x`. For example, `0x00007ffee4b8c000` gives a 64-bit address.

## Process Memory Layout

A typical Unix process address space is organized roughly as follows: ([APUE](#APUE), Sec 7.6)

```txt
              high address
    +-----------------------------+
    |   Command-Line Arguments    |
    | and Environment Variables   |
    +-----------------------------+
    |            Stack            |  <-- grows downward
    |                             |
    +-----------------------------+
    |             ...             |
    +-----------------------------+
    |     Shared Libraries or     |
    |    Memory Mapped Regions    |
    +-----------------------------+
    |             ...             |
    +-----------------------------+
    |             Heap            |  <-- grows upward
    +-----------------------------+
    |    Uninitialized data (bss) |  (initialized to zero by exec)
    +-----------------------------+
    |       Initialized data      |  (read from program file by exec)
    +-----------------------------+
    |            Text             |  (program code)
    +-----------------------------+
               low address
```

In principle, C could modify its own binary code at runtime because the code is a part of the memory as any other data, and hence allow on-the-fly [program modification](https://en.wikipedia.org/wiki/Self-modifying_code).[^1] In practice this is undefined behavior and in modern systems, various security measures are implemented to thwart code modification, called executable space protections.

[^1]: If you wanted to try this out, it's pretty finicky because you have to modify the compiled assembly code, not C code, and setup to override protections in your OS and architecture. There are tutorials from hackers showing how to exploit buffer overflows to inject code, for example.

Likewise, the stack can get overwritten, as happens in buffer overflows. This corrupts the program and potentially allows security hacks.[^2]

[^2]: [*Modified Harvard architecture*](https://en.wikipedia.org/wiki/Modified_Harvard_architecture) is the most common design today for CPUs, it presents the memory model of a single address space for both instructions and data, but in practice they have different caches for instructions and data.

## Stack

### Stack vs Heap

Heap allocation is usually more expensive than stack allocation because of additional bookkeeping and at times having request memory space through OS system calls, which is slower than in-program instructions. 

### Stack frame and functions in C

The call stack is divided into *stack frames*. A function call usually creates a stack frame which may hold the return address for where to go after returning, saved registers and local variables. Values returned are passed from callee to caller using a register to copy the data at hand-off.[^3]

[^3]: Inlining and Return Value Optimizations are about avoiding that data copy and move step.

A function call in another function creates a new stack frame below, not within. That's how recursive functions without end cause stack overflow errors; at some point the memory limit is reached. When a function returns its stack frame is discarded, so local variables stop being available. Returned values are available to the caller in registers and the heap changes persist.

## Address Alignment

Since an object like a 64-bit unsigned integer (`uint64_t`) occupies 8 bytes, advancing a pointer of such elements will increment it by 8. On ARM AArch64 and macOS, `malloc` ([APUE](#APUE), Sec 7.8) returns 16-byte aligned pointers, so their lowest 4 bits are always zero.

Also, many high bits in a 64-bit pointer are unused due to architectural constraints on virtual address space size:

For example, on some 64-bit architectures, only the lower 48 bits (or fewer) are used for virtual addressing. On x86-64, the upper (64 - 48) = 16 bits must follow a special pattern (canonical form): They are sign extensions of bit 47 (the high bit of the virtual address). This means bits 48 through 63 are copies of bit 47, either all zeros or all ones, enforcing address validity.

Because of this regularity, you can save additional meta-data in a pointer value in the high or low bits, and then make the pointer regular again when using it as a reference, 'masking' the meta-data. This is called pointer tagging, a method often used in high-level language implementations for efficient meta-data tracking.

## References

<a id="APUE"></a>
APUE: *Advanced Programming in the UNIX Environment*, 3rd ed., W. Richard Stevens and Stephen A. Rago, Addison-Wesley, 2013.

## Footnotes
