# Project work
Added interception code at client.rs in the server so that I could get the ip commands I made get proccessed first otherwise it processes regular processes first. Once intercepted, the service retrieves the actual system IP address, safely packages it into a fixed-size array to prevent memory panics, and sends it back across the process boundary to the CLI.

# Troubles
Delt with deadlocks and other issues like having to go through how the client.rs and other gate.rs actually worked, so once the DHCP is added this ip command will be able to show all availiable ips. Currently it only shows 1 which is the addr of the system, I can have it also show the mac addr. 

# Future goals
Adding ip addrs this is mainly because I dont want it to show just one ip.
