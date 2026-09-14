# ADR-0003: Initial Hardware Platform

## Status

Proposed

## Context

The microkernel requires an initial hardware target and boot environment
before implementation can begin.

The project's primary goal at this stage is to explore microkernel
architecture, including memory isolation, threads, scheduling,
capabilities, IPC, and userspace services.

Supporting multiple processor architectures, multiprocessor execution,
or implementing a custom bootloader would significantly expand the
initial scope without directly advancing those experiments.

The initial platform should therefore minimize unrelated platform work
while still exposing the hardware mechanisms required to implement and
understand the kernel.

## Decision

### Processor Architecture

The initial kernel will target the x86-64 instruction set architecture.

Architecture-specific mechanisms will be isolated from
architecture-independent kernel code wherever practical so that
additional architectures, such as ARM64, may be supported later.

### Initial Execution Environment

The initial development platform will be QEMU.

The first implementation will assume a single virtual CPU.

Multiprocessor support is explicitly deferred until the fundamental
execution, memory, capability, and IPC models are established.

QEMU is a development environment rather than a permanent architectural
dependency. The kernel should eventually be capable of running on
compatible physical x86-64 hardware.

### Bootloader

The initial kernel will use the Limine bootloader and boot protocol
rather than implement a custom bootloader.

Limine will be responsible for loading the kernel and providing the
initial information required to begin kernel initialization.

The kernel assumes responsibility for system operation from its entry
point onward and should replace bootloader-provided mechanisms with its
own where appropriate.

Implementing a custom bootloader is intentionally deferred.

This decision is based on project scope rather than a belief that
bootloader design is unimportant. The current objective is to reach the
microkernel mechanisms under investigation with as little unrelated
platform machinery as practical.

Once the microkernel architecture is firmly established, bootloading
and early machine initialization may be revisited as a separate area of
study, including the possibility of implementing a custom bootloader.

## Consequences

### Positive

- provides a stable and reproducible x86-64 development target
- allows development to focus on microkernel mechanisms
- avoids premature multiprocessor complexity
- avoids requiring a custom firmware or bootloader implementation
- provides a clear boundary between bootstrapping and kernel operation
- preserves the ability to investigate bootloader design later
- allows architecture-specific code to be identified early

### Negative

- the initial kernel depends on the Limine boot protocol
- early development does not explore firmware and bootloader internals
  in depth
- single-CPU assumptions will eventually need to be revisited
- architecture-specific x86-64 mechanisms will require alternatives
  when additional architectures are supported

## Deferred Work

- custom bootloader implementation
- direct UEFI bootloading
- ARM64 support
- symmetric multiprocessing (SMP)
- physical hardware validation
- deeper investigation of firmware and early machine initialization
