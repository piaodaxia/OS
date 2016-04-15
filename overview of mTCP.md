# mTCP: A Highly Scalable User-level TCP Stack for Multicore Systems

##摘要

缩放TCP短连接的性能是在多核系统上的根本挑战。
虽然许多提案都试图解决各种不足点，但内核执行效率下降现象仍然存在。
例如，即使是最先进的设计，在内核只留下对于用户层创新的小空间的情况下，处理TCP链接需要花70%到80%的CPU周期。
这项工作提出了mTCP,是一个对于多核系统的高性能的用户级TCP协议栈。
mTCP可以解决在包的输入、输出以及管理TCP连接到应用界面时发生的性能下降。
此外，为了采用知名的技术、文中的设计转换多昂贵系统调用一个共享内存的reference；允许有效的flow level事件聚集;以及执行对于高效率输入、输出的输入、输出批量包。
在一个8芯机的评价表明，mTCP在发送小数据的情况下可以挺高性能，它相比于最新的Linux的TCP协议栈3倍，以及相比表现最好的系统研究迄今已知的25个因素。
它在Linux上也可以提高各种有名的应用程序的性能,33%到320%。

##提出的问题

-	缺乏connection locality：存在多线程争用套接字的accept queue的问题，同时，协议栈中处理TCP数据的CPU core不一定是实际上处理数据的应用程序所运行的CPU核。这些都可以导致cache miss或者cache-line sharing问题。
-	共享file descriptor空间：以POSIX为标准，file descriptor在进程共享资源。套接字以file descriptor的形式存在，需要处理multi thread争用问题，同时可以引入file descriptor在VFS中的额外开销。
-	per-packet处理的性能低：socket API的包处理效率低，不能有效地batching。
-	Sys Call开销：BSD socket API对于短连接存在大量user/kernel的切换开销。

##提出的解决方案

mTCP的主要特点如下
-	user-level Packet I/O library：支持event-driven packet I/O接口，基于RSS实现NIC数据的load valancing，避免内核/应用层切换开销；通过利用flow实现CPU解决数据的affinity问题。
-	cache-aware thread placement：优化数据结构，以cache line对齐，并将常访问结构体分量放入相邻区域，减少TLB的不命中性。
-	bached event handling：利用队列对数据包进行批量处理。
-	有效地管理每核的资源：采用lock-free data structure方式，减少lock引入的CPU开销；实现flow-level的线程处理core局部性；进行高效的TCP计时器管理等。
-	短连接优化：基于优先级的包队列，轻量的connection setup。
-	BSD套接字兼容性：易于mTCP的推广部署。

