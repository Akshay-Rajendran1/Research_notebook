Looking into a couple research papers related arrakis:
*  Empowering Cloud Computing With Network Acceleration: A Survey | 2024
  * Talks about how now there are many applications designed to produce timely answers and how there are many new methods being designed for this
  * speaks about how only specialized software like arrakis will be able to help them achive their full potential
  * Shows how processor speed is not able to keep up with ethernet link speed.
  * continue after the graph

# Arrakis summary
Arrakis works by using SR-IOV NICS which stands for single root I/O virtualization. This basically means that the Network interface controller is getting virtualized so that each application thinks it is the only one using it so there is no interaction between them. This is done so that applications can parallely use the nic.
This is done through the help of the kernel for the set up as the kernel allocates it a NIC queue, buffer pool, installs hardware flows and maps this buffer/memmory to the app. The kernel then gives the app pointers to the discriptor ring where each ring contains an array with a pointer to the buffer addr and size along with any flags. It also maps some memmory to it and then gives it the cappability to read/write there. This is only done when the app first requests network acess after that it is no longer needed.
The NIC checks which RX (recieve queue) has what port num and what TX (Send queue) has what port num so that it can send any incoming and outgoing packets through it. 

The difference between what arrakis is doing compared to what I am trying to do in twizzler is:

| Aspect        | Arrakis             | Your Twizzler design     |
| ------------- | ------------------- | ------------------------ |
| Abstraction   | Buffers             | Objects                  |
| Sharing model | Raw pointers        | Object capabilities      |
| Security      | Hardware isolation  | Capability-based access  |

I have not determined how the life cycle would work for these objects yet.

# XDP/eBPF
I was looking into these a little because this is what Linux uses and I wanted to see how another popular os gets over this sort of hardware/kernel limitaion. The basics of this is that there is your code that runs in the kernel which is called eBPF and this basically does anything thing you want it to it is like a customized code you are having run with the kernel for example you could have it do syscalls you want. Overall the kernel is still used here but it is faster. XDP is a type of eBPF and it focuses on redirecting packets so the kernel does not even have to deal with them meanining applications that need speed get speed while the rest of them still go through the kernel. it also can modify these packets and drop them if they are bad or deemed unecessary by the user.

Linux does not use full zero-copy because it would break security features and would require certain applications to be fully rewritten. If the program/app takes care of all this it runs into isssues of if there might be memmory leaks and if the application is slow then there could be backlog and that would cause other issues.
