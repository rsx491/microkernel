# ADR-0001: Kernel Boundary

## Status

Proposed

## Context

The project follows this design philosophy:

> Minimal privileged kernel. Strong process isolation. Extremely cheap communication.

Kernel privilege is treated as a scarce resource. Components should execute in userspace unless they require hardware privilege, protection, scheduling, or fundamental resource arbitration.

## Decision

The kernel will initially be responsible for:

- virtual memory and address-space isolation
- physical memory management
- thread creation and lifecycle
- scheduling
- interrupts and exceptions
- timer facilities
- system call entry
- IPC
- capability/handle management
- minimal hardware abstraction required to support the above

The following should live in userspace wherever practical:

- device drivers
- filesystems
- networking protocols
- process-management policy
- system services
- console services
- higher-level resource policy

## Consequences

### Positive

- smaller trusted computing base
- stronger fault isolation
- easier subsystem replacement
- easier testing of services independently
- lower likelihood that a driver or service failure crashes the kernel

### Negative

- increased IPC traffic
- more context switches
- potentially higher latency
- more complex service orchestration
- performance depends heavily on IPC design

## Open Questions

- capability model vs simpler handle model
- synchronous vs asynchronous IPC
- zero-copy/shared-memory IPC strategy
- whether any drivers should initially remain in kernel space
- process vs thread object model
- syscall ABI size and stability