# My understanding of the current twizzler network stack


General flow (RX)
The physical network card receives an electrical signal and uses DMA to write the raw packet to the OS buffer. The device_thread in net-srv wakes up.
device.rs in net-srv loops through connected apps and copies the packet into the specific application's shared memory ring buffer (rx_buf).
The application's background Engine thread wakes up, seeing new data on its shared memory conveyor belt.
smoltcp reads the shared memory, parses the packet (IP/Port), and sorts the data into the correct internal Socket buffer.
The Waiters system wakes the sleeping user thread. The app thread reads the payload directly from the socket buffer.

General flow (TX)
The user application calls write() on a socket, placing raw data into the smoltcp socket buffer. It calls self.wake() to alert the Engine.
The app's background Engine thread wakes up. smoltcp wraps the raw data in TCP/IP and Ethernet headers to create a full packet.
The Engine pushes the fully formed packet onto the outgoing shared memory conveyor belt (tx_buf) and sends a wake-up signal to net-srv.
Inside net-srv, the dedicated client_thread for this app wakes up. It pulls the packet off the tx_buf queue.
client.rs passes the packet to the virtio_net driver, which instructs the hardware to transmit the data out over the physical wire.

memory management
The lib files handle how the memmroy is allocated as in:
It creates the PacketObject (the shared memory) and the lock-free Queues. 
When the App wants to send data, it asks the library for an empty memory slot (a PacketNum). 
The library manages who owns which piece of memory at any given time so the App and Server don't overwrite each other.
