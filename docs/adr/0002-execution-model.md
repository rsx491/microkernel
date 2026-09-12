# ADR-0002: Execution Model

## Status

Proposed

## Context

ADR-0001 defines the kernel as responsible for thread lifecycle,
scheduling, address-space isolation, and capability management while
keeping process-management policy in userspace.

The execution model therefore needs to define the kernel primitives
used to construct userspace processes without introducing a first-class
kernel process object.

## Decision

A process is not a first-class kernel object.

A process is a userspace abstraction composed from kernel primitives
such as address spaces, threads, and capabilities.

Threads are independently schedulable kernel objects.

Each thread directly references:

- an AddressSpace, defining the memory accessible to the thread
- a CapabilitySpace, defining the kernel objects the thread is
  authorized to use

The kernel does not introduce an additional process object solely to
bind these resources together.

A conventional userspace process domain will normally consist of:

- one AddressSpace
- one CapabilitySpace
- one or more Threads referencing both

The kernel does not require all threads sharing an AddressSpace to also
share the same CapabilitySpace, or vice versa.

### Thread creation

Creating a thread produces an inert, non-runnable kernel object.

Userspace must configure the thread's execution environment before
explicitly making it runnable. This includes, as applicable:

- AddressSpace
- CapabilitySpace
- instruction pointer
- stack pointer
- initial register state

Thread creation and thread execution are therefore separate operations.

### Thread states

The initial execution model is:

INERT
  |
  | start
  v
RUNNABLE
  |
  v
RUNNING
  |
  +---- block/wait ----> BLOCKED
  |                        |
  |                        +---- event ----> RUNNABLE
  |
  +---- exit ----> TERMINATED

The scheduler operates on runnable threads, not processes.

Process-level semantics such as parent/child relationships, executable
identity, process exit codes, and Unix-style zombie processes are
userspace policy and are not inherent kernel thread semantics.

## Consequences

### Positive

- avoids introducing a kernel process abstraction that is not required
  by hardware
- cleanly separates memory isolation from authority
- keeps scheduling centered on the actual executable object: the thread
- allows userspace to construct execution models beyond traditional
  Unix processes
- keeps process-management policy outside the privileged kernel
- inert thread creation avoids execution of partially configured threads

### Negative

- userspace process management becomes more complex
- lifecycle coordination across multiple kernel objects must be handled
  explicitly
- debugging tools cannot assume that a kernel process object exists
- flexible AddressSpace/CapabilitySpace relationships may require clear
  conventions for ordinary applications

## Open Questions

- exact semantics of thread termination and destruction
- whether TERMINATED is a persistent state or only a transient lifecycle
  condition
- how userspace observes thread termination
- ownership and lifetime rules for AddressSpace and CapabilitySpace
- whether threads may switch AddressSpace or CapabilitySpace after start
- initial scheduling policy
