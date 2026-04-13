WEEK 2
I looked into zero copy and the files in twizzler that are related to that, I found that the virtuo net smoltcp files had that kind of information. I am looking at what I have to use for it, and I saw something about Rtokens and T tokens I am not sure what it is about.
I also looked into other research papers related to this and I found arrakis which is something professor owen also told me to look at.
I looked into how zero copy actualy works and it works by directly mapping the memory onto physical space that can also be read by smoltcp so it does not have to make any copies like what is being done right now.
Right now virtuo makes a copy then passes it to smoltcp or it makes a copy from the smoltcps memory spot which leads to higher context overhead for the cpu (zero-copy fixes this).
