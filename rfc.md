Feature Name: zero_copy_net

Start Date: 2026-04-08

#Summary
Implement a networking stack using Twizzler Objects for packet storage. By passing Offsets instead of copying data, we enable zero-copy communication between the hardware, net-srv, and applications.

Motivation
Current networking has "one extra copy" and high context-switch overhead. We need to leverage Twizzler’s shared-memory model to reach line-rate speeds and sub-microsecond latency.

Guide-level explanation
Programmers will interact with a Packet Pool Object.

To Send: Write data into a slot in the Object, then send its Offset to the NIC.

To Receive: The NIC drops data into the Object; you read it directly from that memory.

Two Modes: Use net-srv for shared hardware, or "Direct Mode" for exclusive, high-performance hardware access.

Reference-level explanation
Shared Crate: A twizzler-net-drv crate contains the driver logic, usable by both the system and user apps.

Zero-Copy: We map smoltcp buffers directly onto Twizzler Object memory.

Descriptors: The communication ring uses a simple struct: { object_id: u64, offset: u64, len: u32 }.

Polling: Drivers will spin-wait for packets to reduce wake-up latency.

Drawbacks
Pre-allocating packet objects uses dedicated physical memory.

Direct mode requires strict security checks to prevent apps from crashing the NIC.

Rationale and alternatives
Why? Traditional memcpy-based stacks are too slow for modern 10GbE+ hardware.

Alternative: Standard sockets (slow) or DPDK (complex, non-native).

Prior art
DPDK/SPDK: Proven performance of kernel-bypass and polling.

VirtIO: Standardized ring-buffer descriptors.

Unresolved questions
How do we dynamically resize the Packet Pool if traffic spikes?

Fine-tuning the "spin-before-sleep" timeout duration.

Future possibilities
Hardware-accelerated filtering (e.g., eBPF offloading).

RDMA integration for distributed Twizzler clusters.
