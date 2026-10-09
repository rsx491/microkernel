# ADR-0004: Kernel-Object Lifetime

## Status

Proposed

## Context

ADR-0002 defines an execution model in which threads,
address spaces, and capability spaces are independent
kernel objects.

These objects may be shared between multiple execution
domains and may outlive the thread that originally
created them.

The kernel must define consistent ownership, reference,
and destruction semantics without introducing a
first-class process object.

Object lifetime must remain distinct from the authority
to access an object.

## Decision

### Creator-Independent Lifetime

Creating a kernel object does not establish permanent
ownership over its lifetime.

An object may remain alive after its creator terminates,
provided valid owning references remain.

### Reference Management

The kernel manages references to kernel objects.

Two categories of references contribute to object
lifetime:

1. Capability references
   - Represent authority to access a kernel object.
   - Contribute to its lifetime while valid.

2. Internal kernel references
   - Represent dependencies between kernel objects.
   - Contribute to lifetime independently of capabilities.
   - Do not independently grant userspace authority.

### AddressSpace Lifetime

A thread maintains an internal reference to its
associated AddressSpace.

An AddressSpace remains alive while any thread or
capability maintains an owning reference to it.

Closing the final userspace capability does not destroy
an AddressSpace still referenced by a thread.

### CapabilitySpace Lifetime

A thread maintains an internal reference to its
associated CapabilitySpace.

A CapabilitySpace remains alive while owning references
exist, including references held by threads.

Closing an external capability does not invalidate
a CapabilitySpace still required by a running thread.

### Thread Termination

Thread termination ends execution but does not
automatically destroy shared kernel objects.

A terminating thread releases its internal references
when those dependencies are no longer required.

Kernel objects referenced by other threads or capabilities
remain valid.

Thread termination and thread-object destruction are
distinct lifecycle operations.

### Object Destruction

Kernel objects are reclaimed automatically when:

- No owning capability references remain.
- No owning internal kernel references remain.
- No kernel operation requires the object to remain alive.

Destruction must not invalidate active dependencies.

### Capability Closure and Revocation

Capability closure releases an individual capability
reference.

Capability revocation removes authority according to
revocation semantics defined by the capability model.

Revocation does not necessarily destroy the underlying
kernel object.

Detailed delegation and revocation semantics are
deferred to the capability architecture ADR.

### Reference Cycles

Internal ownership relationships must not create
uncollectable reference cycles.

The kernel must distinguish owning references from
non-owning relationships where necessary.

Reference-counted reclamation must not rely on
tracing garbage collection.

## Rationale

Separating object lifetime from creator lifetime
allows kernel primitives to be shared safely across
userspace execution domains.

Separating capability references from internal kernel
references ensures that authority and lifetime remain
independent concepts.

Automatic reclamation avoids requiring userspace to
manage physical kernel-object destruction directly.

## Consequences

### Positive

- Independent object lifetimes.
- Safe sharing across execution domains.
- Predictable reference-based reclamation.
- Separation of authority and lifetime.
- No requirement for a first-class kernel process.
- Consistent semantics across kernel primitives.

### Negative

- Reference bookkeeping introduces overhead.
- Internal references require carefully defined ownership.
- Reference cycles must be prevented.
- Concurrent reclamation will require synchronization
  when multiprocessor support is introduced.

## Open Questions

- Exact capability revocation semantics.
- Whether revocation affects in-flight operations.
- How userspace observes object destruction.
- Whether thread termination is synchronous.
- Whether TERMINATED persists until thread destruction.
- Detailed synchronization for concurrent reclamation.
- How object destruction failures are handled.
