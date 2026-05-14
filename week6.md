# Week 6

# What I have done:

# New proj

* Spoke to DB and decided to start on an ip utility tool.
* looking at how linux's ip tool works as inspiration/learning how ip utilities work in general.

Reading CHERI paper!

# Summary
What does CHERI do it is a hardware and software design that extends regualr ISA with more capabilities. These capabilities are tokens that can not be made and they replace traditional integer addresses to increase memory protection and making scaling for compartmentalization better. CHERI makes memory unsafe languages more sage and does not increase performace ovehead by much.

# Major Contributions

* Validation tag 1 bit to confirm the validity of the pointer.
* Hybrid model lets the capablities coexist with the MMU based virtual memory and existing C and C++ stacks.
* It uses 128 bit capabilities to fit in 64 bit platforms with precision for small objects.

# Strength/Weaknesses

Strengths - Scalability as it allowes in address space compartmentalization, Strong security since it reduces vunerabilities.

Weaknesses - Poiniter sizes since it goes from a 64 bit to 128 bit size

# Questions
how do unmodified ISAs work?
How will the pointer size affect overheads?

# Project related work 
I made a filter for the cli and I am working on connecting it to smoltcp to get the actual ips, currently I have a dummy code in client.rs which it does reach but I need to figure out how to make it work with smoltcp. The parser uses clap and submodules and derive to make boilerplate things easy then I have the submodules conenct to my addr.rs where the connection to smoltcp takes place, each submodule will connect to a different file this is the one I am working on right now.

# troubles 
I need to spend more time looking into how I can block the rest of the threads connecting to smoltcp so that I can get the ip and then signal to resume it.

# Goals
I am looking forward to making an ip add file (for now I am doing ip addr show first) these two are my goals for now!
