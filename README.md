# Microkernel Manifesto

**Minimal privileged kernel. Strong process isolation. Extremely cheap communication.**

We believe kernel privilege should be treated as a scarce resource. Code belongs in kernel space only when hardware privilege, protection, scheduling, or fundamental resource arbitration requires it.

We believe **failure should be contained**. Drivers, filesystems, networking services, and other system components should execute in isolated address spaces wherever practical. The failure of one service should not imply the failure of the system.

We believe **isolation without efficient communication is incomplete**. IPC is not an auxiliary kernel feature; it is one of the fundamental operations of the system. Communication between isolated components should therefore be designed for extremely low latency, minimal copying, and minimal unnecessary context switching.

We prefer **mechanism over policy in the kernel**. The kernel should provide the primitives necessary to build an operating system—not dictate how every operating-system service must work.

We prefer **small, composable primitives over specialized interfaces**. Every new syscall and privileged abstraction must justify why it cannot be constructed safely and efficiently from mechanisms that already exist.

And ultimately:

**The kernel should protect, schedule, arbitrate, and communicate. Userspace should do everything else it reasonably can.**
