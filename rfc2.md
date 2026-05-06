# Feature Name: twizzler_ip_utility

Start Date: 2026-05-06

# Summary
Implement a native network configuration CLI (ip utility) and control plane for Twizzler OS. By utilizing twizzler-net lock-free queues and shared memory objects, we can query and mutate network state (smoltcp) via net-srv without relying on traditional POSIX sockets or kernel-mediated inter-process communication.

# Motivation
Because Twizzler removes the kernel from the I/O path, standard Linux utilities that rely on NETLINK_ROUTE sockets will not work. We need a native tool to view and manage network interfaces, IP addresses, and routing tables. This utility must align with Twizzler’s data-centric architecture, utilizing the existing object space to read and modify configuration safely.

# Guide-level explanation
Programmers and users will interact with a familiar CLI tool mirroring the Linux iproute2 syntax (e.g., ip addr show).

No Sockets: Instead of opening a socket to talk to a kernel, the utility maps the net-srv control object directly into its address space.

To Read State (e.g., show IPs): The utility pushes a GetInterfaceState command to a lock-free submission queue and rings a doorbell. net-srv reads the internal smoltcp state, copies it to a shared memory buffer, and returns the descriptor via the completion queue. The utility reads this buffer and formats it.

To Mutate State (e.g., add IPs): The utility pushes an AddAddress command to the queue. net-srv safely acquires the lock for smoltcp, applies the new IP, and returns a success/failure code.

Twizzler programmers should think of network configuration as a privileged client submitting RPC-like control descriptors to a background service, rather than streaming byte-serialized requests through a file descriptor.

# Reference-level explanation
Control Plane Queues: We will introduce a dedicated control ring buffer alongside the high-speed packet TX/RX buffers in twizzler-net.

Descriptors: The control communication uses standard enums (ControlCmd and ControlResponse) passed through the ring.

Synchronization: Unlike the data plane which uses spin-waiting for low latency, the control plane will use sys_thread_sync (doorbells) to wake up net-srv, preserving CPU cycles since configuration commands are low-frequency.

CLI Parser: The utility uses the clap crate to handle the nested argument structure (ip [OBJECT] [COMMAND]) natively in Rust.

# Drawbacks
State Locking: Modifying smoltcp state requires net-srv to momentarily lock the network stack, which could stall the high-speed zero-copy data plane during updates.

Security Boundaries: We must ensure strict object permissions so that only authorized user-space applications (like the ip utility) can write to the control submission queue.

Rationale and alternatives
Porting Netlink (Alternative): We could build a POSIX compatibility layer to emulate Netlink sockets. Rationale against: This violates Twizzler's clean-slate, object-based design and introduces massive serialization/deserialization overhead.

Direct Memory Mutation (Alternative): The utility could map the live smoltcp memory with write access and change IPs directly. Rationale against: This would cause race conditions with the net-srv polling loop and potentially crash the NIC drivers.

# Prior art
Linux iproute2: Uses AF_NETLINK sockets. It is highly robust but relies heavily on kernel mediation and complex binary payload parsing.

FreeBSD ifconfig: Uses ioctl system calls on open socket file descriptors.

Both standard OS models rely on the kernel as a bouncer, whereas this RFC treats configuration as a shared-memory client/server interaction.

# Unresolved questions
To resolve before merging:

Buffer Sizing: How do we structure the shared memory completion buffer to handle dynamic-length data (like a massive routing table) without exceeding pre-allocated object limits?

Message Enums: What is the exact set of ControlCmd variants required for a Minimum Viable Product?

During implementation:

How do we ensure net-srv gracefully handles malformed control descriptors without crashing the entire network data plane?

# Out of scope:

Advanced traffic control (like Linux tc) or bridging logic.

# Future possibilities
Extending the control plane to support routing table modifications (ip route).

Allowing applications to subscribe to a "network events" queue to be notified when an interface goes down or an IP changes, without polling.

Deliverables
Week 7-8: A working ip addr show command that successfully maps the control object, pushes a request, and prints live interface state returned by net-srv.

Week 10: Implementation of state mutation (e.g., ip addr add), complete error handling, and a stabilized API in the twizzler-net library.
