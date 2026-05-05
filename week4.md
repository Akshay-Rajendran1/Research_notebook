# Summary
I am looking into how twizzler currently has the NIC interact with devices so that I can learn from it and change it for zero copy. I am also looking at how the polling works, it is quiet impressive.

# Network stack questions/notes
* looking into how srv-io works and if twizzler uses that, it is virualizing the nic, for twizzler it does it based on what the users hardware can handle!
* Twizzler uses polling to check for incoming packets and dma with those packets for things to access it.
* Twizzler does filter the packets with threads as it checks which app requires what then sends it based on the port and header!
* Smol tcp is used to package and unpack incoming/outgoing packets (header and other requirements).
* Engine is the core part that interacts with smoltcp, the memory and the application threads that have been put to sleep/blocked for when there are packets
* Engine thread is woken up when any trip wire for incoming packet is tripped.
* It uses smart pointers...
