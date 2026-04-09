Feature Name: zero_copy_net

Start Date: 2026-04-08

# Summary
Implement a networking stack using Twizzler Objects for packet storage. By passing Offsets instead of copying data, we can use zero-copy communication between the hardware, net-srv, and applications.

# Motivation
Current networking has "one extra copy" and high context-switch overhead. We can use Twizzler's shared-memory model to reach line-rate speeds and reduce latency.

# Guide-level explanation
Programmers will need to interact with a Packet Pool Object.

* The Packet-Pool Object is a Twizzler object where all the network data lives. Instead of the kernel owning the "buffers," you (or net-srv) own a chunk of memory that the hardware can see.
* To Send: Write data into a slot in the Object, then send its Offset to the NIC.
* To Receive: The NIC drops data into the Object; you read it directly from that memory.
* Two Modes: Use net-srv for shared hardware, or "Direct Mode" for exclusive, high-performance hardware access.
* Twizzler programmers should think of networking as managing ownership of shared memory regions within objects rather than streaming bytes through sockets.
* The main difference is that it treats the NIC as another object based client. Currently it uses net-srv and queues to communicate with apps. The current method uses net-srv to hold the packet and make a copy for the smoltcp which is why it is a one copy system now.

# Reference-level explanation
* Zero-Copy: We can map smoltcp buffers directly onto Twizzler Object memory.
* Descriptors: The communication ring uses a simple struct: { object_id: u64, offset: u64, len: u32 }.
* Polling: Drivers will spin-wait for packets to reduce wake-up latency.

# Drawbacks
* Pre-allocating packet objects uses more dedicated physical memory.
* Increased CPU usage because of spin-wait.
* Direct mode requires strict security checks to prevent apps from crashing the NIC.
  * If the app messes up it will require a physical reboot.

# Rationale and alternatives
* Traditional memcpy-based stacks are too slow for modern 10GbE+ hardware.
* Using zero copy helps bypass this bottleneck.
* Standard sockets are (slow) or DPDK (complex, non-native).

# Prior art
* Linux has DPDK which can be used as a bypass tool to let userspace apps talk directly to the NIC.
  * The main issue with this is that it is difficult to set up and it breaks the OS security.
* Arrakis os is another research os that splits the job into os deals with perms and apps deal with data.
  * Similar to Twizzlers net-srv as that checks the perms while apps communicate over the data plane.

# Unresolved questions
* To resolve before merging
  * Descriptor Size: would a 16 byte descriptor be enough for the obj id + offset?
  * What capabilities are required for DMA?
* During implementation
  * Ideal spin-wait time
* Out of scope
  * Encryption
  
# Future possibilities
* Hardware offloaded plugins (eg Checksum offloaded)

