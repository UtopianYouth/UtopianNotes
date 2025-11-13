[TOC]

# 第五章 Linux网络编程

在学习 Linux 网络编程之前，我们需要提前学习计算机网络相关的知识。这里，补充一下在 Linux 网络编程中，用到的一些基础知识（<font color = red>注意</font>：计算机网络是一门值得深入学习的学科，本篇博文只是在宏观的基础上进行计算机网络的介绍）。

## 5.1 网络编程的基础

### 5.1.1 MAC 地址

> 谈到 MAC 地址，离不开计算机硬件：网卡，又称为网络适配器或网络接口卡（Network Interface Card, NIC），属于 OSI 模型的**数据链路层设备**，网卡的主要功能：
>
> - 数据的封装与解封装
> - 链路管理
> - 数据的编码与译码
>
> MAC 地址属于计算机硬件地址，由网络设备制造商烧录在网卡中，MAC地址是**独一无二 48 位（6 字节）串行号**。通常用 12 个 16 进制数表示，如`00-FF-AA-EA-BA-3C`就是一个 MAC 地址，其中，前 3 字节代表网络硬件制造商的编号，由 IEEE 分配，后 3 个字节代表**网络制造商的某个网络产品（如网卡）的系列号**。

### 5.1.2 IP 地址

> <font color = green>简介</font>
>
> IP 地址（Internet Protocol Address）是互联网协议地址，遵循 IPv4 协议，它的最大贡献是，**屏蔽了数据链路层不同网络设备的 MAC 地址差异**。IP 地址是 IP 协议提供的一种统一的地址格式，为网络层中的每一台互联网设备分配一个逻辑地址。
>
> IP 地址是一个 32 位的二进制数，通常用点分十进制表示，如`192.168.1.1`是一个 IP 地址。
>
> <font color = green>IP 地址编址方式</font>
>
> 每一个 IP 地址包括两个标识码（ID），即网络 ID 和主机 ID。同一个物理网络上的所有主机都使用同一个网络 ID，网络上的一个网络设备有一个主机 ID 与其对应。Internet 委员会定义了 5 种 IP 地址类型以适合不同容量的网络，即 A 类 ~ E 类。**其中A、B 和 C 类网络在全球范围内统一分配**，D、E类为特殊地址。
>
> | 类别 | 最大网络数    | IP 地址范围                 | 单个网段最大主机数 | 私有 IP 地址范围              |
> | ---- | ------------- | --------------------------- | ------------------ | ----------------------------- |
> | A    | 126(2^7-2)    | 1.0.0.1 ~ 126.255.255.255   | 16777214(2^24 - 2) | 10.0.0.0 ~ 10.255.255.255     |
> | B    | 16384(2^14)   | 128.0.0.1 ~ 191.255.255.254 | 65534(2^18-2)      | 172.16.0.0 ~ 172.31.255.255   |
> | C    | 2097152(2^21) | 192.0.0.1 ~ 223.255.255.254 | 254(2^8-2)         | 192.168.0.0 ~ 192.168.255.255 |
>
> <font color = green>A 类 IP 地址</font>
>
> A 类 IP 地址，第一个字节为网络号，剩下三个字节为主机号。**规定 A 类网络地址，最高位必须是 0（解释一下，规定最高位的值，可以在没有子网掩码的前提下，区分 A、B 和 C 类网络）**。
>
> - A 类 IP 地址范围是 1.0.0.1 ~ 126.255.255.254（二进制表示，自行转换），其中，主机号全 1 表示广播地址，如：1.255.255.255 表示在网络 1.0.0.0 种广播。
> - A 类 IP 地址的子网掩码是 255.0.0.0。
>
> <font color = green>B 类 IP 地址</font>
>
> B 类 IP 地址，前两个字节为网络号，剩下的两个字节为主机号。**规定 B 类网络地址，最高两位必须是 10**，B 类网络地址**适合中等规模的网络**。
>
> - B 类 IP 地址范围是 128.0.0.1 ~ 191.255.255.254（二进制表示，自行转换），主机号全 1 表示广播地址。
> - B 类 IP 地址的子网掩码为 255.255.0.0。
>
> <font color = green>C 类 IP 地址</font>
>
> C 类 IP 地址，前三个字节为网络号，剩下的一个字节为主机号。**规定 C 类网络地址，最高三位必须是 110**，C 类网络适合**小规模的局域网络**。
>
> - C 类 IP 地址范围是 192.0.0.1 ~ 223.255.255.254（二进制表示，自行转换），主机号全 1 表示广播地址。
> - C 类 IP 地址的子网掩码为 255.255.255.0。
>
> <font color = green>D 类 IP 地址</font>
>
> D 类 IP 地址在历史上被叫做**多播地址（multicast address），也叫组播地址**。在以太网中，多播地址命名了一组站点（接收方），该组站点**接收发送方发送的多播数据**。**多播地址最高位必须是 1110**。
>
> - D 类 IP 地址范围是 224.0.0.0 ~ 239.255.255.255
>
> <font color = green>一些特殊的 IP 地址</font>
>
> - IP 地址中，以 11110 开头的 E 类 IP 地址，都保留用于将来和实验使用。
>
> - 127.0.0.1 ~ 127.255.255.255 用于回路测试，如：127.0.0.1 表示本机 IP 地址。

### 5.1.3 子网掩码

> 子网掩码（subnet mask）又叫做网络掩码、地址掩码，它的作用是指明 IP 地址，**哪些位标识主机所在的子网，以及哪些位标识主机的位掩码**。子网掩码不能单独存在，**它必须结合 IP 地址一起使用**，划分 IP 地址，得到网络号和主机号。
>
> 子网掩码是一个 32 位的地址，用于屏蔽 IP 地址的一部分，**以区别网络标识和主机标识**。

### 5.1.4 端口

> <font color = green>简介</font>
>
> 端口（port）可以简单的理解为网络设备与外界通讯交流的出口，可以分为虚拟端口和物理端口，其中，**虚拟端口指计算机内部或交换机路由器内的端口**，不可见，比如 TCP/IP 协议中的端口、计算机中的 80 端口、21 端口和 23 端口等。物理端口又称为接口，是可见端口，如 RJ45 网口等。
>
> 如果把 IP 地址比作一间房子，端口就是出入这间房子的门。真正的房子只有几个门，但是一个 IP 地址的端口，可以有 65536(2^16) 个之多，**端口是通过端口号来标记的，端口号由两字节的无符号整数表示**。
>
> <font color = green>端口类型</font>
>
> <font color = red>1.周知端口（Well Known Ports）</font>
>
> 周知端口，也叫知名端口或者常用端口，**范围从 0 到 1023**，它们紧密绑定于一些特定的服务。例如 80 端口分配给 WWW 服务，21端口分配给 FTP 服务，23 端口分配给 Telnet 服务等等。在我们日常生活中，在浏览器输入域名是不用指定端口号的，因为默认情况下，WWW 服务的端口号是 80。**网络服务也可以使用其它端口号，比如使用 8080 作为 WWW 服务的端口，则需要在地址栏里输入"域名: 8080"**。还有些**系统协议使用的固定端口号，也属于周知端口**，比如 139 端口专门用于 NetBIOS 与 TCP/IP 之间的通信。
>
> <font color = red>2.注册端口（Registered Ports）</font>
>
> **端口号从 1024 到 49151**，它们松散地绑定于一些服务，**分配给用户进程**，这些进程主要是用户**选择安装的一些应用程序**，而不是已经分配好了公认端口的常用程序。
>
> <font color = red>3.动态端口/私有端口（Dynamic Ports / Private Ports）</font>
>
> 动态端口范围是从 49152 到 65535。之所以称为动态端口，是因为它一般不固定分配某种服务，而是动态分配。
>
> 

## 5.2 网络模型

### 5.2.1 OSI七层参考模型

> 七层模型，**亦称 OSI（Open System Interconnection）参考模型**，即开放式系统互联。七层参考模型是国际标准组织（ISO）制定的一个，用于计算机或通信系统间互联的标准体系。它是一个七层的、抽象的模型体，不仅包括一系列抽象的术语或概念，也包括具体的协议。
>
> <center>
> <img src = "./images/第五章 Linux网络编程/5-4 OSI七层参考模型.png" width = "80%">
> </center>
>
> <font color = green>物理层</font>，主要定义物理设备标准，如网线的接口类型、光纤的接口类型、各种传输介质的传输速率等。它的**主要作用是传输比特流（就是由数字信号 1、0 转换为电流强弱来进行传输，到达数据接收方后，再由电流强弱转化为 1、0数字信号，也可以理解为数模转化与模数转换） **，物理层的数据叫做**比特**。
>
> <font color = green>数据链路层</font>，**建立逻辑连接、进行硬件地址寻址、差错校验等功能**。定义了如何让格式化数据**以帧为单位进行传输，以及如何控制物理介质的访问**。将比特流组合成字节进而组合成帧，用 MAC 地址访问介质。链路层传输的数据通常叫**MAC帧**。
>
> <font color = green>网络层</font>，进行逻辑地址寻址，在位于不同地理位置的网络中的两个主机系统之间，提供连接和路由选择。Internet 的发展，使得**从世界各站点访问信息**的用户数大大增加，而网络层正是管理这种连接的层。网络层传输的数据通常叫**IP数据报**。
>
> <font color = green>传输层</font>，定义了一些**传输数据的的协议和端口号（WWW 端口 80 等）**，如：
>
> - TCP：传输控制协议，**传输效率低，可靠性强（等待确认机制）**，主要用于传输可靠性要求高，数据量大的数据。
> - UDP：用户数据报协议，**传输效率高，非可靠传输**，主要用于传输可靠性要求不高，数据量小的数据。
>
> 传输层还负责**将从下层接收的数据，进行分段和传输**，到达目的地后，再进行重组。传输层传输的数据通常叫**TCP/UDP报文段**。
>
> <font color = green>会话层</font>，通过传输层（端口号：传输端口与接收端口）建立数据传输的通路。**会话层主要用于发起会话和接收会话请求**。
>
> <font color = green>表示层</font>，该层主要是对接收的数据**进行解释、加密与解密、压缩与解压缩等（也可以简单理解为，将计算机能够识别的东西转换为人能够识别的东西：图片、声音等）**。
>
> <font color = green>应用层</font>，网络服务与最终用户的一个接口（用户通过该接口使用网络服务），**应用层为用户的应用程序（例如电子邮件、文件传输和终端仿真）提供网络服务**。只有连接了 Internet 的应用程序，**才能通过网络接口使用网络服务**。

### 5.2.2 TCP/IP 四层模型

> 目前，Internet（因特网）使用的主流协议族是 TCP/IP 协议族，它是一个分层，多协议的通信体系。TCP/IP 协议族是一个四层协议系统，自低而上，分别是数据链路层、网络层、传输层和应用层。**每一层完成不同的功能，且通过若干协议来实现，上层协议使用下层协议提供的服务**。
>

<center>
<img src = "./images/第五章 Linux网络编程/5-4 TCP&IP协议族.png" width = "80%">
</center>

>
> TCP/IP 四层模型参考了 OSI 七层模型，**并且对其进行了简化**。
>
> - OSI七层模型中，应用层、会话层和表示层，三个层次提供的服务相似，在 TCP/IP 协议中，**统一为应用层**。
> - 传输层和网络层在网络协议中的地位十分重要，在 TCP/IP 协议中，作为独立的两层。
> - 数据链路层和物理层内容差不多，在 TCP/IP 协议中，被归并为**网络接口层**。四层体系结构的 TCP/IP 协议，**在实际应用中效率更高，成本更低**。
>

<center>
  <img src = "./images/第五章 Linux网络编程/5-4 OSI模型和TCP&IP模型.png" width = "70%">
</center>


> <font color = green>应用层</font>，是 TCP/IP 协议的第一层，直接为应用进程提供服务。
>
> - 对于不同的应用程序，它们会根据自己的需求**来使用应用层的不同协议**，邮件传输应用使用了 SMTP 协议、万维网使用了 HTTP 协议、远程登录服务使用了 TELNET 协议。
> - 应用层还能**加密、解密、格式化数据**。
> - 应用层可以建立或解除与其它节点的联系，这样可以充分节省网络资源。
>
> <font color = green>传输层</font>，作为 TCP/IP 协议的第二层。传输层在整个 TCP/IP 协议中，很重要，主要提供端到端的通信。
>
> <font color = green>网络层</font>，属于 TCP/IP 协议中的第三层。主要进行网络连接的建立和终止，以及 IP 地址的寻址等功能。
>
> <font color = green>网络接口层</font>，属于 TCP/IP 协议中的第四层。**网络接口层既是传输数据的物理媒介，也为网络层提供了一条准确无误的线路**。

## 5.3 协议

<font color = green>简介：</font>**协议是网络协议的简称**，网络协议是通信双方**必须共同遵守的一组规则**。如怎样建立连接、怎样互相识别等。只有遵守这个规则，计算机之间才能相互通信交流，这套规则就统称为**协议**。协议的三要素：<font color = red>语法、语义和时序</font>。协议最终体现为**在网络上传输数据包的格式**。

### 5.3.1 TCP/IP 模型中常见的协议

> **应用层常见的协议有**：FTP协议（File Transfer Protocol 文件传输协议）、HTTP协议（Hyper Text Transfer Protocol 超文本传输协议）和NFS（Network File System 网络文件系统）。
>
> **传输层常见的协议有**：TCP协议（Transmission Control Protocol 传输控制协议）、UDP协议（User Datagram Protocol 用户数据报协议）。
>
> **网络层常见的协议有**：IP协议（Internet Protocol 因特网互联协议）、ICMP协议（Internet Control Message Protocol 因特网控制报文协议）、IGMP协议（Internet Group Management Protocol 因特网组管理协议）。
>
> **网络接口层常见协议有**：ARP协议（Address Resolution Protocol 地址解析协议）、RARP协议（Reverse Address Resolution Protocol 逆地址解析协议）。

### 5.3.2 UDP 协议

<center>
  <img src = "./images/第五章 Linux网络编程/5-5 UDP协议1.png" width = "80%"
</center>


> - 源端口号：发送方端口号。
> - 目的端口号：接收方端口号。
> - 长度：UDP 用户数据报的长度，最小值为 8 bytes（只有首部）。
> - 校验和：检测 UDP 用户数据报在传输中是否有错，有错就丢弃。

### 5.3.3 TCP 协议

<center>
  <img src = "./images/第五章 Linux网络编程/5-5 TCP协议1.png" width = "85%">
</center>


> 1. 源端口号：发送方端口号。
> 2. 目的端口号：接收方端口号。
> 3. 序列号：本报文段的数据的**第一个数据字节的序号**。
> 4. 确认序号：期望收到对方**下一个报文段的第一个字节的序号**。
> 5. 首部长度（数据偏移）：TCP 报文段的起始处，距离 TCP 报文段的数据起始处有多少单位长度，即首部长度。单位：**32位，以 4  字节为计算单位**。4 位最大表示值为 15 个单位长度，即 TCP 首部的长度最大为 60 bytes。 
> 6. 保留：占 6 位，保留为今后使用，目前应置为 0。
> 7. 紧急 URG：此位置为 1，表示紧急字段有效，它告诉系统**此 TCP 报文段中有紧急数据，应尽快传送**。
> 8. 确认 ACK：仅当 ACK  = 1 时确认号字段才有效，TCP 规定，在连接建立后**所有传达的 TCP 报文段都必须把 ACK 置 1**，表示已经收到了发送方发送的数据。
> 9. 推送 PSH：当两个应用进程进行交互式的通信时， 有时发送端的应用进程**希望在键入一个命令后，立即就能够收到接收端的响应**。在这种情况下，TCP 就可以使用推送（push）操作，这时，发送端 TCP 把 PUSH 置 1，并立即创建一个 TCP 报文段发送出去，接收方收到 PSH = 1 的报文段，**就尽快地（即”推送“前进）交付给接收应用进程，而不再等到整个缓存都填满后再向上交付**。
> 10. 复位 RST：用于复位相应的 TCP 连接
> 11. 同步 SYN：仅在三次握手建立 TCP 连接时有效，当 SYN = 1 而 ACK = 0时，表示这是一个 TCP 连接请求报文段，对方若同意建立连接，则应在相应的报文段中使用 SYN = 1和 ACK = 1。所以，**SYN = 1 就表示这是一个连接请求或连接接受 TCP 报文段**。
> 12. 终止 FIN：用来断开 TCP 连接。当 FIN = 1 时，表明此 TCP 报文段的发送方已经发送完数据，并要求断开 TCP 连接。
> 13. 窗口大小：表示发送方接收窗口的大小。
> 14. 校验和：校验和字段**检验的范围包括首部和数据两部分**，在计算校验和时**需要加上 12 字节的伪首部**。
> 15. 紧急指针：仅仅在 URG = 1 时才有意义，它指出了**本 TCP 报文段中的紧急数据的字节数（紧急数据结束后就是普通数据）**，即指出了紧急数据的末尾在 TCP 报文段中的位置（注意：即使接收方的接受窗口大小为 0 时，发送方也可以发送紧急数据）。
> 16. 选项：长度可变，最长可达 40 字节，当没有使用选项时，**TCP 首部长度是 20 字节**。

### 5.3.4 IP 协议

<center>
  <img src = "./images/第五章 Linux网络编程/5-5 IP协议1.png" width = "90%">
</center>


> 1. 版本号：IP 协议的版本。通信双方使用过的 IP 协议的版本必须一致。目前最广泛使用的 IP 协议版本号为 4（即 IPv4）。
> 2. 首部长度：首部长度，**单位是 32 位（4 字节）**。4 位最大表示值为 15 个单位长度，即 IP 首部的长度最大为 60 bytes。
> 3. 服务类型：一般不适用，取值为 0。
> 4. 总长度：**指首部加上数据**的总长度，单位为字节。
> 5.  标识（identification）：IP 软件在存储器中维持一个计数器，每产生一个数据报，计数器就加 1，并将此值赋值给标识。
> 6. 标志（flag）：目前只有两位有意义
>    - 标志字段中，最低位为 MF，MF = 1 即表示后面**还有分片**的数据报。MF = 0 表示这已是**若干数据报片中的最后一片**。
>    - 标志字段中，中间一位为 DF，DF = 1 表示**不允许 IP 数据报分片**，DF = 0 表示**允许 IP 数据报分片**。
> 7. 片偏移：指出较长的 IP 数据报在分片后，**某一片在源 IP 数据报的相对位置，片偏移以 8 字节为偏移单位**。
> 8. 生存时间（Time To Live, TTL）：表明 IP 数据报在网络中的寿命。即为**跳数限制**，由发出 IP 数据报的源点设置这个字段。路由器在转发数据之前，就把 TTL 的值减一，当 TTL 的值减为 0 时，就丢弃这个 IP 数据报。
> 9. 协议：指出此数据报携带的数据使用何种协议，以便**使目的主机的 IP 层知道应将数据部分上交给哪个处理过程**，常用的协议如：ICMP(1)，IGMP(2)，TCP(6)，UDP(17)，IPv6(41)。
> 10. 首部校验和：只校验数据报的首部，**不包括数据部分（数据部分传输层已经进行了校验）**。
> 11. 源地址：发送方的 IP 地址。
> 12. 目的地址：接收方的 IP 地址。

### 5.3.5 以太网帧协议

<center>
  <img src = "./images/第五章 Linux网络编程/5-5 MAC帧协议1.png" width = "90%">
</center>


> 1. 类型：0x800 表示 IP，0x806 表示 ARP，0x835 表示 RARP。
>
> 2. CRC（Cycle Redundancy Check 循环冗余检验）：通过 CRC 检测以太网帧的所有字段，来发现以太网帧中是否有比特错误、多比特错误或突发错误等。

### 5.3.6 ARP 协议

<center>
  <img src = "./images/第五章 Linux网络编程/5-5 ARP协议1.png" width = "90%">
</center>


> 1. 硬件类型：1 表示 MAC 地址。
> 2. 协议类型：0x800 表示 IP 地址。
> 3. 硬件地址长度：6（MAC 地址）。
> 4. 协议地址长度：4（IP 地址）。
> 5. 操作：1 表示 ARP 请求，2 表示 ARP 应答，3 表示 RARP 请求，4 表示 RARP 应答。

### 5.3.7 封装

> 计算机网络TCP/IP模型中，上层协议是如何使用下层协议提供的服务的呢？**其实就是通过封装（encapsulation）实现的**。应用程序的数据发送到物理网络上之前，**将沿着协议栈从上往下依次传递**。每层协议都将在上层数据的基础上，**加上自己的头部信息（有时还包括尾部信息）**，以实现该层的功能，这个过程就是封装。

<center>
  <img src = "./images/第五章 Linux网络编程/5-6 计算机网络对数据的封装1.png" width = "90%">
</center>


### 5.3.8 分用

> 当 MAC 帧到达目的主机时，**将沿着协议栈自底向上依次传递**，各层协议依次处理 MAC 帧中本层负责的头部数据，以获取所需的信息，并最终将处理后的 MAC 帧交给目标应用程序，这个过程称为**分用（demultiplexing）**，分用是**依靠头部信息中的类型字段实现的**。

<center>
  <img src = "./images/第五章 Linux网络编程/5-6 以太网帧分用的过程1.png" width = "90%">
</center>


## 5.4 网络通信过程

在网络通信的过程中，就是对数据的**一系列封装和解封装过程**。

<center>
  <img src = "./images/第五章 Linux网络编程/5-6 网络通信过程1.png" width = "90%">
</center>


## 5.5 socket 介绍

> socket（套接字），就是对网络中**不同主机上的应用进程之间进行双向通信的端点的抽象**。一个 socket 就是网络上进程通信的一端，提供了应用层的**进程利用网络协议交换数据的机制**。从所处的地位来讲，socket 上联应用进程，下联网络协议栈，**是应用程序通过网络协议进行通信的接口，也是应用程序与网络协议根进行交互的接口**。
>
> socket 可以看成是计算机网络中，各自通信连接中的端点，这是一个逻辑上的概念。它是网络环境中，进程间通信的 API，**也是可以被命名和寻址的通信端点**。在通信的主机中，**每一个 socket 都有其类型，并且有一个与之相连的进程**。通信时，其中一个网络应用程序**将要传输的一段信息写入它所在主机的 socket 中**，该 socket 通过与网络接口卡（NIC）相连的传输介质将这段信息送到另外一台主机的 socket 中，使对方能够接收到这段信息。**socket 是由 IP 地址和 port 结合的，提供向应用层进程传送数据包的机制**。
>
> 在 Linux 环境下，socket 的本质为**内核借助缓冲区形成的伪文件**，既然是文件，**就可以使用文件描述符引用 socket**。与管道类似，Linux 内核将 socket 封装成文件的目的是**为了统一接口**，使得读写套接字和读写文件的操作一致。区别是管道主要应用于本地进程间通信，而 socket 应用于网络进程间数据的传递。
>
> <font color = green>套接字通信分为两部分</font>
>
> - 服务器端：**被动**接受客户端的连接请求。
> - 客户端：**主动**向服务器发送连接请求。
>
> socket 是一套通信的接口，Linux 和 Windows 都有，只存在一些细微的差别。

## 5.6 字节序

现代 CPU 的累加器一次都能装载（至少）4 字节（32 位）的数据，那么这 4 字节内容**在内存中的排列顺序**将影响它被累加器**装载成数据的真实值**，这就是字节序的问题。在不同的计算机体系结构中，对于字节、字等的存储机制有所不同，因而引发了**计算机通信领域中的一个很重要问题**，即通信双方交流的信息单元（比特、字节、字、双字等等）应该以什么样的顺序进程传递。如果不达成一致的规则，通信双方将无法进行正确的解码 / 编码，从而导致通信失败。

> <font color = red>**字节序，顾名思义就是字节的顺序，就是大于一个字节的数据在内存中的存放顺序（一个字节的数据就不用讨论字节序了）。**</font>
>
> 在常见的计算机体系结构中，字节序分为大端字节序（Big - Endian）和小端字节序（Little - Endian）。大端字节序是指**数据的最高位字节存放在低地址处**，小端字节序是指**数据的最低位字节存放在低地址处**。
>
> - 小端字节序（以一个 8 字节类型的数据`0x1122334412345678`为例）
>
> <center>
>   <img src = "./images/第五章 Linux网络编程/5-9 小端字节序.png" width = "90%">
> </center>
>
>
> - 大端字节序（以一个 8 字节类型的数据`0x1234567811223344`为例）
>
> <center>
>   <img src = "./images/第五章 Linux网络编程/5-9 大端字节序.png" width = "90%">
> </center>

## 5.7 网络通信字节序转换

当格式化的数据在两台**使用不同字节序的主机之间**直接传递时，接收端必然会对数据解码错误。一个解决方案是：发送端总是采用大端字节序发送，接收端知道发送端采用的是大端字节序。

**网络字节序**是 TCP/IP 规定好的一种数据表示格式，**它与具体的 CPU 类型、操作系统等无关**，从而可以保证数据在不同主机之间**能够被正确解释**，网络字节序采用的是**大端排序方式**。

### 5.7.1 字节序转换函数

BSD(Berkeley Software Distribution) socket 提供了**封装好的字节序转换接口**，方便程序员使用。包括从主机字节序到网络字节序的转换函数：`htons(); htonl();`，从网络字节序到主机字节序的转换函数：`ntohs(); ntohl();`。

```c
// 在进行网络通信时，需要将主机字节序转换为网络字节序（大端）
#include <arpa/inet.h>
/*
	字节序转换函数
 	- h：host，主机字节序
  - to：从...转换成...
  - n：network，网络字节序
  - s：unsigned short 类型序列
  - long：unsigned int 类型序列
*/

/*
	函数功能：将 unsigned short 类型变量（端口）从主机字节序转换为网络字节序
  函数参数
  - hostshort：需要转换的无符号短整型数（端口）
  返回值：转换后的结果
*/
uint16_t htons(uint16_t hostshort);

/*
	函数功能：将 unsigned short 类型变量（端口）从网络字节序转换为主机字节序
  函数参数
  - hostshort：需要转换的无符号短整型数
  返回值：转换后的结果
*/
uint16_t ntohs(uint16_t netshort);

/*
	函数功能：将 unsigned int 类型变量（IP地址）从主机字节序转换为网络字节序
  函数参数
  - hostshort：需要转换的无符号整型数
  返回值：转换后的结果
*/
uint32_t htonl(uint32_t hostlong);

/*
	函数功能：将 unsigned int 类型变量（IP地址）从网络字节序转换为主机字节序
  函数参数
  - hostshort：需要转换的无符号整型数
  返回值：转换后的结果
*/
uint32_t ntohl(uint32_t netlong);
```

### 5.7.2 字节序转换函数使用案例

```c
#include<stdio.h>
#include<arpa/inet.h>
#include<stdlib.h>

int main() {
    unsigned short num1 = 0x0102;
    // 主机字节序转换为网络字节序
    unsigned short num2 = htons(num1);
    // 网络字节序转换为主机字节序
    unsigned short num3 = ntohs(num2);

    printf("source num = %x, htons() num = %x, ntohs() num = %x\n", num1, num2, num3);

    unsigned char buf1[4] = { 192,168,0,100 };
    // 定义两个指针，接收转换后的IP地址
    unsigned int* buf2 = (unsigned int*)malloc(sizeof(unsigned int));
    unsigned int* buf3 = (unsigned int*)malloc(sizeof(unsigned int));

    // 主机字节序转换为网络字节序
    *buf2 = htonl(*((uint32_t*)buf1));
    // 网络字节序转换为主机字节序
    *buf3 = ntohl(*buf2);

    // 无符号整数，printf() 的 format 为 %u
    unsigned char* p = (char*)buf1;
    printf("source IP: %u.%u.%u.%u\n", *p, *(p + 1), *(p + 2), *(p + 3));

    p = (char*)buf2;
    printf("htonl() IP: %u.%u.%u.%u\n", *p, *(p + 1), *(p + 2), *(p + 3));

    p = (char*)buf3;
    printf("ntohl() IP: %u.%u.%u.%u\n", *p, *(p + 1), *(p + 2), *(p + 3));

    return 0;
}
```

## 5.8 socket 地址

在 Linux 内核中，socket 地址其实是一个结构体，封装了端口号和IP等信息。

### 5.8.1 通用 socket 地址

socket 网络编程接口中，表示 socket 地址的是结构体`sockaddr`，其定义如下：

```c
#include<bits/socket.h>
struct sockaddr{
  sa_family_t sa_family;	// 无符号短整型
  char sa_data[14];
};

typedef unsigned short int sa_family_t;
```

`sa_family`是一个成员变量，表示地址族类型。**地址族类型通常与协议族类型对应**。常见的协议族（protocol family，也称 domain）和对应的地址族如下所示：

| 协议族     | 地址族     | 描述              |
| ---------- | ---------- | ----------------- |
| `PF_UNIX`  | `AF_UNIX`  | UNIX 本地域协议族 |
| `PF_INET`  | `AF_INET`  | TCP/IPv4 协议族   |
| `PF_INET6` | `AF_INET6` | TCP/IPv6 协议族   |

宏 `PF_*` 和 `AF_*`都定义在`bits/socket.h`头文件中，且后者与前者有完全相同的值，所以二者通常混用。`sa_data`成员用于存放 socket 地址的值。**不同协议族的地址值具有不同的含义和长度**，如下所示：

| 协议族     | 地址值含义和长度                                             |
| ---------- | ------------------------------------------------------------ |
| `PF_UNIX`  | 文件的路径名，长度可达到 108 字节                            |
| `PF_INET`  | 16 bits 的端口号和 32 bits IPv4地址，共 6 字节               |
| `PF_INET6` | 16 bits 的端口号，32 bits 流标识，128 bits IPv6 地址，32 bits 范围 ID，共 26 字节 |

由上表可知，`sockaddr`结构体中，14 字节的`sa_data`根本**无法容纳多数协议族的地址值**。因此，Linux 内核定义了下面这个新的 socket 地址结构体，这个结构体不仅提供了足够大的空间用于存放地址值，而且是**内存对齐的**。

```c
#include<bits/socket.h>
struct sockaddr_storage{
  sa_family_t sa_family;	// 无符号短整型
  unsigned long int __ss_align;
  char __ss_padding[128 - sizeof(__ss_align)];
};

typedef unsigned short int sa_family_t;
```

### 5.8.2 专用 socket 地址

很多网络编程函数诞生早于 IPv4 协议，那时候都使用的是`struct sockaddr`结构体，为了向前兼容，**现在`sockaddr`退化成了`(void*)`的作用（针对函数传参，传递任何数据类型的地址）**，至于 socket 地址对象是`sockaddr_in`还是`sockaddr_in6`，由地址族确定， 然后函数内部再强制类型转换为所需的地址类型。

<center>
  <img src = "./images/第五章 Linux网络编程/5-11 专用socket地址1.png" width = "80%">
</center>


UNIX 本地域协议族使用如下专用的 socket 地址结构体：

```c
#include<sys/un.h>
struct sockaddr_un{
  sa_family_t sin_family;
  char sun_path[108];
};

typedef unsigned short int sa_family_t;
```

TCP/IP 协议族有 `sockaddr_in`和 `sockaddr_in6`两个专用的 socket 地址结构体，它们分别用于 IPv4 和 IPv6：

```c
#include<netinet/in.h>

struct sockaddr_in {
    sa_family_t sin_family;  	/* 地址族, AF_INET */
    in_port_t sin_port;    		/* 端口号, 网络字节序 */
    struct in_addr sin_addr; 	/* IPv4 地址 */
    char  sin_zero[8]; 				/* 填充字节，用于对齐 */
};

struct in_addr {
  uint32_t s_addr;     
};

struct sockaddr_in6 {
    sa_family_t sin6_family;   	/* 地址族, AF_INET6 */
    in_port_t sin6_port;     		/* 端口号, 网络字节序 */
    uint32_t sin6_flowinfo; 		/* 流量信息 */
    struct in6_addr sin6_addr;  /* IPv6 地址 */
    uint32_t sin6_scope_id; 		/* 范围 ID */
};

struct in6_addr {
    union {
        uint8_t u6_addr8[16];    /* 16 个字节的数组 */
        uint16_t u6_addr16[8];   /* 8 个 16 位的短整数 */
        uint32_t u6_addr32[4];   /* 4 个 32 位的整数 */
    } in6_u;
};

typedef unsigned short uint16_t;
typedef unsigned int uint32_t;
typedef uint16_t in_port_t;		// 基于 IPv4 的 port 
typedef uint32_t in_addr_t;		// 基于 IPv4 的 ip 地址
```

所有**专用的 socket 地址（以及 socket storage）类型的变量在实际使用时都要转换为通用的 socket 地址（强制类型转换即可）**，因为所有 socket 编程接口使用的**地址参数（形参列表）**类型都是`sockaddr`。

## 5.9 IP 地址转换函数

通常，人们习惯用可读性好的字符串来表示 IP 地址，比如用点分十进制字符串表示 IPv4 地址，以及用十六进制字符串表示 IPv6 地址。但在编程中，**我们需要先把它们转换为整数（二进制）**方能使用。而记录日志时则相反，**我们要把整数表示的 IP 地址转换为可读的字符串形式**。

### 5.9.1 IP 地址转换函数介绍

```c
#include <arpa/inet.h>

// 下面两个API中，p 表示点分十进制的 IP 字符串，n 表示网络字节序

/*
    函数功能：将点分十进制的 IP 字符串转换为网络字节序的二进制形式
    函数参数
    - af：需要转换协议族（地址族），常见的 AF_INET( IPv4 ) 和 AF_INET6( IPv6 )
    - src：需要转换的 IP 地址字符串
    - dst：存储转换后网络字节序的二进制结果
    返回值
    - 转换成功：返回 1
    - 转换失败：返回 0，表示 src 对于 af 协议族来说，不是一个有效的 IP 地址
    - 转换失败：返回 -1，表示 af 不是一个有效的 IP 地址协议族

*/
int inet_pton(int af, const char *src, void *dst);

/*
    函数功能：将网络字节序的二进制 IP 地址形式转换为字符串类型的 IP 地址形式
    函数参数
    - af：需要转换协议族（地址族），常见的 AF_INET( IPv4 ) 和 AF_INET6( IPv6 )
    - src：需要转换的二进制网络字节序
    - dst：存储转换后的 IP 文本字符串结果
    - size：dst 字符数组的大小
    返回值
    - 转换成功：返回转换后 IP 字符串的地址，和 dst 的值一样
    - 转换失败，返回 NULL，并且设置错误号
*/
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```

### 5.9.2 IP 地址转换函数使用案例

```c
#include<stdio.h>
#include<stdlib.h>
#include<arpa/inet.h>

int main(int argc, char* argv[]) {
    // 点分十进制 IP 转换为网络字节序二进制
    char src_ip[] = "192.168.1.3";
    unsigned int num = 0;
  
    inet_pton(AF_INET, src_ip, (void*)&num);
    unsigned char* p = (char*)&num;
    printf("%d.%d.%d.%d\n", *p, *(p + 1), *(p + 2), *(p + 3));

    // 网络字节序二进制转换为点分十进制 IP, 4*3 + 3 + 1 = 16
    char* dst_ip = (char*)malloc(16 * sizeof(char));
  
    const char* dst_ip_addr = inet_ntop(AF_INET, (void*)&num, dst_ip, 16);
    printf("%s\n", dst_ip);
    printf("%d\n", dst_ip == dst_ip_addr);

    return 0;
}
```

## 5.10 TCP 通信流程

1. 首先，介绍一下 **TCP 和 UDP 的区别**，它们都属于传输层协议：

|                        | UDP                                | TCP                        |
| ---------------------- | ---------------------------------- | -------------------------- |
| 是否创建连接           | 否                                 | 是                         |
| 是否可靠               | 否                                 | 是                         |
| 进行通信的连接对象个数 | 一对一、一对多、多对一和多对多     | 一对一                     |
| 传输方式               | 面向 UDP 数据报                    | 面向字节流                 |
| 首部开销               | 8 字节                             | 最少 20 字节               |
| 适用场景               | 实时应用，速度要求高（视频会议等） | 可靠性高的应用（文件传输） |

2. TCP 通信流程**总览**

<center>
  <img src = "./images/第五章 Linux网络编程/5-13 TCP通信流程1.png" width = "70%">
</center>


> <font color = green>服务器端</font>
>
> - 创建一个用于监听的 socket
>   - 监听：监听客户端的连接
>   - socket：在 Linux 内核中，socket 通过文件描述符表示
> - 将这个监听的文件描述符**和本地的（IP + port）绑定（服务器的地址信息）**
>   - 客户端通过本地的（IP + port）与服务器进行连接
> - 设置监听，**监听的文件描述符开始工作**，监听的文件描述符**指向的内存缓冲区有内容，说明有新的客户端发起连接请求**。
> - 阻塞等待，**当有客户端向服务器发起连接，解除阻塞**，接受客户端的连接，会得到一个**和客户端通信的 socket（这个 socket 存储了客户端的地址信息，并且以文件描述符的形式表示）**
> - 通信
>   - 接收数据
>   - 发送数据
> - 通信结束，断开连接
>
> <font color = green>客户端</font>
>
> - 创建一个用于通信的 socket
> - 连接服务器，需要指定连接服务器的 IP 和 port
> - 连接成功，客户端直接与服务器通信
>      - 接收数据
>      - 发送数据
> - 通信结束，断开连接

## 5.11 socket 函数

```c
#include <sys/types.h>
#include <sys/socket.h>

/*
    函数功能：创建一个套接字，用于网络通信
    函数参数
    - domain：TPC/IP 协议族（地址族），常见的AF_INET(IPv4), AF_INET6(IPv6), AF_LOCAL(local communication)
    - type：通信过程中使用的协议类型
        - SOCK_STREAM：提供一个有序的、可靠的、双向通信的、基于连接的字节序协议类型（TCP）
        - SOCK_DGRAM：提供一个面向无连接的、不可靠的、任意数据长度的数据报式（datagrams）协议类型（UDP）
        ...
    - protocol：使用具体的协议，一般写 0 即可
        - SOCK_STREAM：流式协议默认使用 TCP
        - SOCK_DGRAM：报式协议默认使用 UDP
    返回值
    - 成功：返回一个该 socket 的文件描述符
    - 失败：返回 -1，并且设置错误号，具体参考 linux 源码说明，man 2 socket
*/
int socket(int domain, int type, int protocol);

/*
    函数功能：将创建好的 socket 文件描述符（fd）和本地的 IP 地址 + 端口进行绑定（也叫 socket 命名）
    函数参数
    - sockfd：通过 socket() 函数得到的文件描述符
    - addr：需要绑定的 socket 地址，这个地址封装了 ip 和端口号的信息
    - addrlen：指定 addr 所指向的 socket 结构体大小
    返回值
    - 成功：返回 0
    - 失败：返回-1，并且设置错误号，具体参考 linux 源码说明，man 2 bind
*/
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

/*
    函数功能：监听 sockfd 绑定的 socket 上的连接
    函数参数
    - sockfd：通过 socket() 函数得到的文件描述符
    - backlog：监听队列的最大长度，即在调用 accept 之前可以排队未完成连接请求的数量，pending connection：待连接
        - 一般的，使用 cat /proc/sys/net/core/somaxconn 查看系统支持的最大监听队列长度
    返回值
    - 成功：返回 0
    - 失败：返回-1，并且设置错误号，具体参考 linux 源码说明，man 2 listen
*/
int listen(int sockfd, int backlog);

/*
    函数功能：接收客户端发来的网络通信连接请求，该函数会阻塞，等待客户端连接
    函数参数
    - sockfd：用于监听的文件描述符
    - addr：socket 地址，传出参数，记录了连接成功后，客户端的地址信息（ip，port）
    - addrlen：第二个参数结构体所占内存的大小
    返回值
    - 成功：用于通信的文件描述符（每一个客户端请求连接，连接成功后，都会返回一个文件描述符）
    - 失败：返回 -1，并且设置错误号，具体参考 linux 源码说明，man 2 accept
*/
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);

/*
    函数功能：客户端连接服务器
    函数参数
    - sockfd：用于与客户端通信的文件描述符
    - addr：客户端要连接的服务器地址信息
    - addrlen：第二个参数 addr 所占内存的大小
    返回值
    - 成功：返回 0
    - 失败：返回 -1，并且设置错误号，具体参考 linux 源码说明，man 2 connect
*/
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

```

通过 socket 相关的函数，客户端与服务器端连接成功后，就可以进行通信了，常见的网络 socket 间通信方式如下（和进程间通信差不多）：

```c
/*
    网络通信基础方式，向缓冲区中读写数据
    #include <unistd.h>

    ssize_t write(int fd, const void *buf, size_t count);
    函数功能：向缓冲区写数据
    函数参数
    - fd：需要写数据缓冲区的文件描述符
    - buf：缓冲区所在内存的地址
    - count：写入数据的字节数
    返回值
    - 成功：返回成功写入数据的字节数
    - 失败：返回 -1，并且设置错误号，具体参考 linux 源码说明，man 2 write

    ssize_t read(int fd, void *buf, size_t count);
    函数功能：从缓冲区读数据
    函数参数
    - fd：读缓冲区的文件描述符
    - buf：缓冲区所在内存的地址
    - count：读数据的字节数
    返回值
    - 成功：返回成功读取数据的字节数
    - 失败：返回 -1，并且设置错误号，具体参考 linux 源码说明，man 2 read
*/
```

### 5.11.1 TCP 通信实现（服务器端）

```c
#include<stdio.h>
#include<arpa/inet.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>

int main() {
    // 1. 创建 socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);    // tcp
    if (listen_fd == -1) {
        perror("create socket...");
        exit(0);
    }

    // 2. 将 socket 对应的 fd 绑定 IP 地址和 port
    struct sockaddr_in socket_addr;
  
  	// 初始化协议族
    socket_addr.sin_family = AF_INET;  
  
  	// 服务器端所有网络接口的 IP 都可用于监听客户端的连接
    socket_addr.sin_addr.s_addr = INADDR_ANY;
  
  	// 设置服务器端监听的端口号
    socket_addr.sin_port = htons(9999);         

    int ret1 = bind(listen_fd, (struct sockaddr*)&socket_addr, sizeof(socket_addr));
    if (ret1 == -1) {
        perror("bind...");
        exit(0);
    }

    // 3. 监听
    int ret2 = listen(listen_fd, 10);
    if (ret2 == -1) {
        perror("listen...");
        exit(0);
    }
    // 4. 接收客户端发来的连接请求
    struct sockaddr_in client_addr;     // 存储客户端的 socket 信息
    socklen_t len = sizeof(client_addr);    // 存储客户端的 socket 所占内存大小
    int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &len);     // 在有客户端申请连接之前，会阻塞
    if (communication_fd == -1) {
        perror("accept...");
    }
    printf("client connection successfully...\n");

    // 输出客户端的信息
    char client_ip[16] = { '\0' };
    inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
    unsigned short client_port = ntohs(client_addr.sin_port);
    printf("client ip = %s, client port = %d\n", client_ip, client_port);


    // 5. 通信，获取客户端数据，给客户端发送数据
    char receive_buf[1024] = { 0 };
    while (1) {
        int ret4 = read(communication_fd, receive_buf, sizeof(receive_buf));
        if (ret4 > 0) {
            printf("receive_client_buf: %s\n", receive_buf);
        }
        else if (ret4 == 0) {
            // 在 TCP 通信中，等于 0 的情况表示客户端断开连接
            printf("client closed...\n");
            break;
        }
        else if (ret4 == -1) {
            perror("read...");
            exit(0);
        }

        write(communication_fd, receive_buf, sizeof(receive_buf));
    }

    // 6. 关闭文件描述符，断开 TCP 连接
    close(listen_fd);
    close(communication_fd);

    return 0;
}
```

### 5.11.2 TCP 通信实现（客户端）

```c
#include<stdio.h>
#include<arpa/inet.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>

int main() {
    // 1. 创建 socket
    int communication_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (communication_fd == -1) {
        perror("socket...");
        exit(0);
    }
    // 2. 连接服务器端，创建 sockaddr_in 对象
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    inet_pton(AF_INET, "192.168.10.1", &server_addr.sin_addr.s_addr);
    server_addr.sin_port = htons(9999);
    int ret1 = connect(communication_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));

    if (ret1 == -1) {
        perror("connect");
        exit(0);
    }

    // 3. 通信
    char receive_buf[1024] = { '\0' };
    while (1) {
        sleep(2);
        char* send_data = "hello, communication info...";
        write(communication_fd, send_data, strlen(send_data));  // 向服务器端发送数据

        int ret2 = read(communication_fd, receive_buf, sizeof(receive_buf));    // 读取服务器端发送的数据
        if (ret2 > 0) {
            printf("receive_server_buf: %s\n", receive_buf);
        }
        else if (ret2 == 0) {
            // 在 TCP 通信中，等于 0 的情况表示服务器端断开连接
            printf("server closed...\n");
            break;
        }
        else if (ret2 == -1) {
            perror("read...");
            exit(0);
        }
    }
    // 4. 关闭文件描述符，断开 TCP 连接
    close(communication_fd);

    return 0;
}
```

## 5.13 TCP 三次握手与四次挥手

TCP 是一种面向连接的单播协议，在发送数据前，通信双方必须在彼此间建立一条连接。所谓的**连接**，其实是客户端和服务器的内存里保存的一份关于对方的信息，如 IP 地址、端口号等等。

TCP 可以看成是一种字节流，它会处理 IP 层或以下的层的丢包、重复以及错误问题。在连接的建立过程中，双方需要交换一些连接的参数。这些参数可以放在 TCP 头部。

TCP 提供了一种**可靠、面向连接、字节流、传输层**的服务，采用三次握手建立一个连接，采用四次挥手关闭一个连接。

<center>
  <img src = "./images/第五章 Linux网络编程/5-5 TCP协议1.png" width = "90%">
</center>


> 1. 紧急 URG：此位置为 1，表示紧急字段有效，它告诉系统**此 TCP 报文段中有紧急数据，应尽快传送**。
> 2. 确认 ACK：仅当 ACK  = 1 时确认号字段才有效，TCP 规定，在连接建立后**所有传达的 TCP 报文段都必须把 ACK 置 1**，表示已经收到了发送方发送的数据。
> 3. 推送 PSH：当两个应用进程进行交互式的通信时， 有时发送端的应用进程**希望在键入一个命令后，立即就能够收到接收端的响应**。在这种情况下，TCP 就可以使用推送（push）操作，这时，发送端 TCP 把 PUSH 置 1，并立即创建一个 TCP 报文段发送出去，接收方收到 PSH = 1 的报文段，**就尽快地（即”推送“前进）交付给接收应用进程，而不再等到整个缓存都填满后再向上交付**。
> 4. 复位 RST：用于复位相应的 TCP 连接
> 5. 同步 SYN：**仅在三次握手建立 TCP 连接时有效**，当 SYN = 1 而 ACK = 0时，表示这是一个 TCP 连接请求报文段，对方若同意建立连接，则应在相应的报文段中使用 SYN = 1和 ACK = 1。所以，**SYN = 1 就表示这是一个连接请求或连接接受 TCP 报文段**。
> 6. 终止 FIN：**断开 TCP 连接**。当 FIN = 1 时，表明此 TCP 报文段的发送方已经发送完数据，并要求断开 TCP 连接。

### 5.13.1 TCP 三次握手

> - 三次握手的目的是**保证双方互相之间建立了连接**。
>
> - 三次握手发生在**客户端连接的时候**，当客户端调用`connect()`，底层会通过 TCP 协议进行三次握手。

<center>
  <img src = "./images/第五章 Linux网络编程/5-17 TCP三次握手1.png" width = "90%">
</center>

### 5.13.2 TCP 四次挥手

> - TCP 四次挥手发生在断开连接的时候，当在程序中调用了`close()`会使用 TCP 协议进行四次挥手。
> - 客户端和服务器端**都可以主动发起断开连接**，谁先调用`close()`谁就发起断开连接请求。
> - 因为在 TCP 连接的时候，采用三次握手建立的连接是**双向的**，所以在断开的时候**需要双向断开**。
> - 

<center>
  <img src = "./images/第五章 Linux网络编程/5-19 TCP四次挥手1.png" width = "90%">
</center>


## 5.14 滑动窗口

滑动窗口（sliding window）是一种流量控制技术（基于端到端）。早期的网 通信中，通信双方**不会考虑网络的拥挤情况**，而是直接发送数据。由于大家都不知道网络拥塞的情况，同时发送数据，导致中间节点阻塞掉包，谁也发不了数据，所以就有了**滑动窗口机制**来解决此问题。

TCP 中采用滑动窗口来进行传输控制，滑动窗口的大小意味着**接收方有多大的缓冲区**可以用于接收数据。发送方可以**通过滑动窗口的大小来确定应该发送多少字节的数据**。当滑动窗口为 0 时，除了紧急标志位 `URG = 1` 的数据，发送方一般不能再发送 TCP 报文段。

滑动窗口是 TCP 中实现诸如 ACK 确认、流量控制和拥塞控制的承载结构。

<center>
  <img src = "./images/第五章 Linux网络编程/5-18 TCP滑动窗口1.png" width = "90%">
</center>


> **上图中的信息解释如下**：
>
> <font color = green>发送方进程</font>
>
> - 白色格子：表示发送方滑动窗口剩余的空间
> - 灰色格子：表示发送方已经发送出去的数据，但是还没有被接收
> - 粉色格子：表示发送方待发送的数据
>
> <font color = green>接收方进程</font>
>
> - 白色格子：表示接收方滑动窗口剩余的空间
> - 粉色格子：已经接收到的数据

<center>
  <img src = "./images/第五章 Linux网络编程/5-18 TCP滑动窗口2.png" width = "95%">
</center>


> 上图表明了，客户端和服务器端进行 TCP 三次握手后，进行通信时，**各自滑动窗口大小的变化**：
>
> - 图的右边展示了**服务器端滑动窗口的大致情况。**
> - mss 表示 Maximum Segment Size（TCP 报文段的最大字节数）。
> - win 表示数据发送方（可以是客户端也可以是服务器端），接收窗口的大小。
> - 2049(1024) 表示，发送字节序的起始位置为 2049 ，发送的字节数为 1024。
>
> <font color = green>通信序号解释</font>：
>
> - 序号 1：客户端向服务器发起 TCP 连接请求，并且告诉服务器，客户端的滑动窗口大小是 4096，一次发送 TCP 报文段的最大字节数是 1460，<font color = red>第一次握手</font>；
> - 序号 2：服务器接受客户端的 TCP 连接请求，并且向客户端发送 TCP 连接请求，告诉客户端，服务器端的滑动窗口大小是 6144，一次发送 TCP 报文段的最大字节数是 1024，<font color = red>第二次握手</font>；
> - 序号 3：客户端发送确认报文段，接受服务器端的 TCP 连接请求，<font color = red>第三次握手</font>；
> - 序号 4 ~ 9：客户端向服务器发送了 6k 的数据，并且每次都告诉服务器，滑动窗口大小是 4096；
> - 序号 10：服务器告诉客户端，6k 数据已经接收到，存储到滑动窗口缓冲区中，并且缓冲区的数据已经处理了 2k，滑动窗口大小是 2k；
> - 序号 11：服务器告诉客户端，6k 数据已经接收到，存储到滑动窗口缓冲区中，并且缓冲区的数据已经处理了 4k，滑动窗口大小是 4k；
> - 序号 12：客户端给服务器发送了 1k 的数据，并且告诉服务器，滑动窗口大小是 4096；
> - 序号 13：客户端主动请求和服务器断开连接，并且给服务器发送了 1k 的数据，<font color = red>第一次挥手</font>；
> - 序号 14：服务器回复 **ACK = 8194**（因为客户端发送 FIN = 1 会消耗一个字节序），<font color = red>第二次挥手</font>；
>   - 同意客户端的断开连接请求
>   - 告诉客户端已经接收到刚才发的 2k 数据
>   - 告诉客户端滑动窗口大小是 2k
> - 序号 15：告诉客户端滑动窗口大小是 4k；
> - 序号 16：告诉客户端滑动窗口大小是 6k；
> - 序号 17：服务器给客户端发送 FIN = 1 报文段，请求断开连接，<font color = red>第三次挥手</font>；
> - 序号 18：客户端同意服务器端的断开连接请求，<font color = red>第四次挥手</font>。

## 5.15 多进程实现并发服务器

要实现 TCP 通信**服务器处理并发任务**（处理多个客户端发送 TCP 连接请求），**使用多线程或者多进程解决**。

> **多进程实现思路**：
>
> - 一个父进程，多个子进程。
>- 父进程负责等待并且**接受客户端的连接请求**。
> - 子进程完成通信，接收一个客户端的连接，就**创建一个子进程**。

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<arpa/inet.h>
#include<signal.h>
#include<wait.h>
#include<errno.h>

/*
    关于子进程资源如何回收的问题
    - 多进程实现 TCP 通信服务器端，由于父进程一直处于运行状态（不断监听新的 TCP 连接）,而子进程只负责通信，当通信结束后，子进程也结束运行，但是子进程资源一直没有被释放。
        - 回收方法1：在父进程逻辑中调用 wait 函数，不可取，因为 wait 阻塞会影响监听。
        - 回收方法2：在父进程逻辑中调用 waitpid函数，也不可取，因为 accept 函数会阻塞，影响子进程回收。
        - 回收方法3：使用信号机制，一旦有子进程运行结束，父进程捕捉信号，回收子进程资源，可取。
*/
void recycle_child_process(int arg) {
    while (1) {
        // 调用一次 waitpid 回收一个子进程资源
        int ret = waitpid(-1, NULL, WNOHANG);
        if (ret == -1) {
            // 所有子进程都被回收了
            break;
        }
        else if (ret == 0) {
            // 子进程正在运行
            break;
        }
        else if (ret > 0) {
            printf("recycle child process, pid = %d\n", ret);
        }
    }
}

int main(int argc, char* argv[]) {

    // 0. 使用信号机制，回收子进程资源
    struct sigaction act;
    act.sa_flags = 0;   // 使用 act.sa_handler 对应的函数
    sigemptyset(&act.sa_mask);      // 清空信号集掩码，表示不临时阻塞任何信号
    act.sa_handler = recycle_child_process;
    sigaction(SIGCHLD, &act, NULL); // 注册信号捕捉

    // 1. 创建监听客户端连接用的 socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2. 给监听客户端的 socket 绑定 IP 和 port
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
  
  	// 指定服务器监听的端口号
    server_addr.sin_port = htons(9999);   
  
  	// 服务器端所有网络接口的 IP 都可用于监听客户端的连接
    server_addr.sin_addr.s_addr = INADDR_ANY;       
    int ret1 = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(-1);
    }

    // 3. 监听客户端发来的连接请求
    int ret2 = listen(listen_fd, 10);
    if (ret2 == -1) {
        perror("listen");
    }

    pid_t pid = 0;

    // 4. 父进程不断接受客户端发来的 TCP 连接
    while (1) {
        struct sockaddr_in client_addr;
        socklen_t len = (socklen_t)sizeof(client_addr);

        // 该函数会阻塞，直到有客户端发来了连接请求（监听文件描述符对应的 socket 监听到 TCP 连接），才会执行
        int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &len);
        if (communication_fd == -1) {
            if (errno == EINTR) {
                // accept 系统调用会被捕捉到的信号中断
                continue;
            }
            perror("accept");
            exit(-1);
        }
        printf("client connect success.\n");

        pid = fork();
        if (pid > 0) {
            // 父进程
        }
        else if (pid == 0) {
            // 获取客户端信息
            char client_ip[16] = { '\0' };
            unsigned short client_port = 0;
            inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
            client_port = ntohs(client_addr.sin_port);
            printf("pid = %d, client_ip = %s, client_port = %u.\n", getpid(), client_ip, client_port);

            // 5. 子进程与客户端进行通信
            while (1) {
                char receive_data[1024] = { '\0' };
                int ret3 = read(communication_fd, receive_data, sizeof(receive_data));
                if (ret3 == 0) {
                    // 客户端断开连接
                    printf("client closed.\n");
                    break;
                }
                else if (ret3 > 0) {
                    printf("pid = %d, receive client data: %s\n", getpid(), receive_data);
                    write(communication_fd, receive_data, sizeof(receive_data));
                }
                else {
                    perror("read");
                    exit(-1);
                }
            }

            // 6. 子进程关闭通信文件描述符
            close(communication_fd);
            exit(0);
        }
        else {
            perror("fork");
            break;
        }
    }

    // 7. 父进程释放监听用的文件描述符
    if (pid > 0) {
        close(listen_fd);
    }
    return 0;
}
```

## 5.16 多线程实现并发服务器

**多线程实现思路**

> - 主线程创建监听 TCP 连接用的 socket
>- 监听客户端发送的连接请求
> - 接受客户端发送的连接请求
> - 子线程进行通信
> - 释放监听和通信使用的文件描述符资源

```c
// 方法1，考虑到子线程需要获取的数据，使用了结构体，较为复杂
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
#include<arpa/inet.h>
#include<string.h>

// 子线程和客户端通信必要的 info
typedef struct SocketInfo {
    pthread_t tid;                      // 线程id
    struct sockaddr_in client_addr;     // 客户端的 socket 信息
    int communication_fd;               // 通信使用的 fd
}SocketInfo;

SocketInfo socket_infos[128];           // 限制客户端连接的个数

// 线程与客户端通信
void* thread_working(void* arg) {

    SocketInfo* socket_info = (SocketInfo*)arg;
    while (1) {
        // 获取客户端信息
        char client_ip[16];
        inet_ntop(AF_INET, (struct sockaddr*)&socket_info->client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
        unsigned short port = ntohs(socket_info->client_addr.sin_port);

        // 接收客户端发来的数据
        char receive_buf[1024] = { '\0' };
        int ret = read(socket_info->communication_fd, receive_buf, sizeof(receive_buf));
        if (ret == 0) {
            // 客户端断开连接
            printf("client closed\n");
            break;
        }
        else if (ret == -1) {
            perror("read");
            break;
        }
        else if (ret > 0) {
            printf("receive data of client ip = %s, port = %d: %s\n", client_ip, port, receive_buf);
        }
        write(socket_info->communication_fd, receive_buf, sizeof(receive_buf));
    }

    // 通信结束，关闭文件描述符资源，将使用的 socket_info 资源初始化，socket_info 属于全局变量
    close(socket_info->communication_fd);
    socket_info->communication_fd = -1;
    socket_info->tid = -1;

    return NULL;
}

int main(int argc, char* argv[]) {
    // 创建 socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(0);
    }

    // 绑定 IP + PORT
    struct sockaddr_in server_addr;
  
  	// 指定服务器端所有的网络接口 IP 用于监听客户端的连接
    server_addr.sin_addr.s_addr = INADDR_ANY;  
  	
  	// 指定协议族类型
    server_addr.sin_family = AF_INET;
  
  	// 指定服务器监听的端口号
    server_addr.sin_port = htons(9999);

    int ret1 = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(0);
    }

    // 监听客户端发来的连接请求
    int ret2 = listen(listen_fd, 128);
    if (ret2 == -1) {
        perror("listen");
        exit(0);
    }

    // 初始化结构体数据
    int max_len = sizeof(socket_infos) / sizeof(socket_infos[0]);
    for (int i = 0;i < max_len;++i) {
        memset(socket_infos + i, 0, sizeof(socket_infos[i]));       // 初始化内存区域为 0
        socket_infos[i].communication_fd = -1;
        socket_infos[i].tid = -1;
    }

    // 利用多线程技术，并发处理客户端发来的连接请求
    while (1) {
        struct sockaddr_in client_addr;     // 存储客户端的 socket 信息
        socklen_t len = sizeof(client_addr);
        int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &len);
        if (communication_fd == -1) {
            perror("accept");
            exit(0);
        }

        // 创建子线程，用于处理客户端的连接请求
        SocketInfo* arg;
        for (int i = 0;i < max_len;++i) {
            // 从数组中找到一个可以用的 SocketInfo 元素
            if (socket_infos[i].communication_fd == -1) {
                arg = socket_infos + i;
                break;
            }
            if (i == max_len - 1) {
                sleep(1);
                i = 0;      // 如果 128 个通信的 fd 都在使用，从头开始，每隔 1s，等待空闲的 fd 出现
            }
        }

        arg->communication_fd = communication_fd;
        memcpy(&arg->client_addr, &client_addr, len);   // 结构体赋值

        pthread_create(&arg->tid, NULL, thread_working, arg);
        // 线程函数执行结束，释放线程资源
        pthread_detach(arg->tid);

    }

    close(listen_fd);
    return 0;
}
```

```c
// 方法2，考虑子线程只需要获取通信用的文件描述符，代码量减少
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
#include<arpa/inet.h>
#include<string.h>

// 子线程逻辑
void* child_pthread(void* arg) {
    int communication_fd = *((int*)arg);
    // 与客户端进行通信
    while (1) {
        char receive_buf[1024] = { '\0' };
        int ret = read(communication_fd, receive_buf, sizeof(receive_buf));
        if (ret == 0) {
            // 客户端断开 TCP 连接
            printf("client close.\n");
            break;
        }
        else if (ret == -1) {
            perror("read");
            close(communication_fd);
            exit(-1);
        }
        else {
            printf("tid = %ld receive client data: %s\n", pthread_self(), receive_buf);
            write(communication_fd, receive_buf, sizeof(receive_buf));
        }
    }
    close(communication_fd);
    return NULL;
}

int main(int argc, char* argv[]) {
    // 1. 创建 socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(0);
    }

    // 2. 绑定 IP + PORT
    struct sockaddr_in server_addr;
  
  	// 指定所有网络接口的 IP 监听客户端的连接
    server_addr.sin_addr.s_addr = INADDR_ANY;      
  	
  	// 指定协议族
    server_addr.sin_family = AF_INET;
  
  	// 指定服务器端的监听端口号
    server_addr.sin_port = htons(9999);

    int ret1 = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(0);
    }

    // 监听客户端发来的连接请求
    int ret2 = listen(listen_fd, 128);
    if (ret2 == -1) {
        perror("listen");
        exit(0);
    }

    // 利用多线程技术，并发处理客户端发来的连接请求
    while (1) {
        struct sockaddr_in client_addr;     // 存储客户端的 socket 信息
        socklen_t len = sizeof(client_addr);
        int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &len);
        if (communication_fd == -1) {
            perror("accept");
            exit(0);
        }
        printf("connect success.\n");

        // 获取客户端的信息
        char client_ip[16] = { '\0' };
        unsigned short client_port = ntohs(client_addr.sin_port);
        inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
        printf("client ip = %s, client port = %u.\n", client_ip, client_port);

        // 创建子线程，用于处理客户端的连接请求
        pthread_t tid;
        pthread_create(&tid, NULL, child_pthread, &communication_fd);

        // 设置线程分离
        pthread_detach(tid);
    }
  
    // 关闭监听用的文件描述符
    close(listen_fd);
    return 0;
}
```

## 5.17 TCP 状态转换

<center>
  <img src = "./images/第五章 Linux网络编程/5-23 TCP状态转换3.png" width = "90%">
</center>


> 上图中，展示了 TCP 通信的整个周期，**从连接、通信、再到断开连接的状态转换**，其中，左边表示客户端，右边表示服务器端。
>
> <font color = green>2MSL (最大报文段生存时间，Maximum Segment Lifetime)</font>
>
> - 在 TCP 连接释放四次挥手中，主动断开连接的一方，最后处于 `TIME_WAIT` 状态，这个状态会持续 2MSL，是为了**保证被动关闭 TCP 连接的一方能够正常关闭 TCP 连接**。
>   - MSL：官方建议 2 分钟，Linux 内核是 30s。
>
> <font color = red>为什么会是 2MSL 呢？</font>
>
> 当 TCP 连接主动关闭方接收到被动关闭方**发送的 FIN 和最终的 ACK **后，TCP 连接的主动关闭方必须处于 `TIME_WAIT` 状态，并且持续 2MSL。
>
> 这样就能保证 TCP 连接的主动关闭方在它**发送最终的 ACK 丢失的情况下，重新发送最终的 ACK。**
>
> TCP 连接主动关闭方重新发送最终的 ACK 是因为**它上一次发送的最终 ACK 在网络中丢失了，被动关闭方由于没有接收到主动关闭方的 ACK，导致被动关闭方需要重发 FIN。**事实上，如果被动关闭方**一直没有收到主动关闭方的 ACK**，它就会一直重传 FIN 报文段，主动关闭方也会在收到 FIN 报文段后，**重新发送最终的 ACK**。
>
> 综上，由于被动关闭方在 MSL 时间内，没有收到主动关闭方发送的最终 ACK，然后被动关闭方重发 FIN。经过 MSL 时间后，主动关闭方收到了被动关闭方重发的 FIN，又会重新发送最终的 ACK。所以是 2MSL。
>
> <font color = red>注意</font>：主动关闭方重新发送最终的 ACK 后，`TIME_WAIT` 阶段**又会重新计时为 2MSL**。

## 5.18 TCP 连接的半关闭状态

当 TCP 连接中 A 向 B 发送 FIN 请求关闭，另一端 B 回应 ACK 之后（A 端进入 `FIN_WAIT2`状态），并没有立即发送 FIN 给 A，**A方处于半连接状态（半开关）**。此时 A 可以接收 B 发送的数据，但是 A 已经不能再向 B 发送数据。

从程序的角度来看，**可以使用 API 来实现半连接状态**：

```c
#include <sys/socket.h>

/*
    函数功能：关闭文件描述符 sockfd 绑定的 socket 对应的全双工通信连接
    函数参数
    - sockfd：需要关闭的 socket 的 fd
    - how：允许为 shutdown 操作选择以下几种方式
        - SHUT_RD: 对应 0，关闭 sockfd 上的读功能，此选项不允许 sockfd 进行读操作，并且，该套接字不再接收数据。
        任何当前在套接字，接收缓冲区的数据将被无声的丢弃掉（sockfd 是缓冲区对应的文件描述符，套接字在缓冲区中操作数据）
        - SHUT_WR: 对应 1，关闭 sockfd 的写功能，此选项不允许 sockfd 进行写操作，进程不能在对此套接字发出写操作。
        - SHUT_RDWR: 对应2，关闭 sockfd 的读写功能，相当于调用 shutdown 两次，首先是 SHUT_RD，然后是 SHUT_WR。
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号，具体参考 man 2 shutdown
*/
int shutdown(int sockfd, int how);
```

> 使用 `close()` 终止一个连接，但它只是减少文件描述符的引用计数， 只有当文件描述符的引用计数为 0 时，才关闭 TCP 连接。 `shutdown()` 不考虑文件描述符的引用计数，直接关闭文件描述符。也可选择终止一个方向的连接，只终止读或只终止写。
>
> <font color = red>注意：</font>
>
> 1. 如果有多个进程共享一个 socket，`close()`每被调用一次计数减 1，直到计数为 0 时，**也就是所有进程都调用了 `close()`，socket 将被释放**。
>
> 2. 在多进程中如果一个进程调用了 `shutdown(sfd, SHUT_RDWR)` 后，其它进程将无法进行通信。但如果一个进程调用了 `close(sfd)` 将不会影响到其它进程。

## 5.19 端口复用

端口复用最常用的用途是：

> - 防止服务器重启时之前绑定的端口还未释放；
> - 程序突然退出而系统没有释放端口。
>

```c
#include <sys/types.h>
#include <sys/socket.h>

/*
    函数功能：获取套接字选项的系统调用，查询与指定套接字相关的各种配置参数
    函数参数
    - sockfd：需要查询套接字的fd
    - level：选项所在的协议级别，通常为 SOL_SOCKET（端口复用级别）
    - optname：要查询具体选项的名称
        - SO_RCVBUF（接收缓冲区大小）
        - SO_SNDBUF（发送缓冲区大小）
    - optval：optname 取值所在的地址（optname，有些取值是 int 类型，有些是特定的结构体类型）
    - optlen：optval 缓冲区长度
    返回值
    - 成功：返回 0
    - 失败：返回 -1，并且设置错误号
*/
int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);

/*
    函数功能：设置套接字属性（不仅仅能设置端口复用）
    函数参数
    - sockfd：需要操作 socket 的文件描述符
    - level：所设置套接字属性的协议及别，和 getsockopt() 一样，通常为 SOL_SOCKET（端口复用级别）
    - optname：需要设置选项的名称
        - SO_REUSEADDR：地址复用
        - SO_REUSEPORT：端口复用
    - optval：optname 取值所在的地址（optname，有些取值是 int 类型，有些是特定的结构体类型）
        - 当 optname 是 SO_REUSEPORT 时
            - *optval = 1：表示可以端口复用
            - *optval = 0：表示不可以端口复用
    - optlen：optval 参数的大小
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号
*/
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

/*
    一般地，端口复用设置的时机是在服务器绑定端口之前
    setsockopt();
    bind();
*/
```

```c
// 基于 socket 通信的特殊 API
#include <sys/types.h>
#include <sys/socket.h>

/*
*/
ssize_t recv(int sockfd, void *buf, size_t len, int flags);

/*
*/
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);

/*
*/
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

## 5.20 I/O 多路复用

**I/O 多路复用（I/O多路转接）使得程序能同时监听多个文件描述符，也可以理解为在服务器端的程序，可以并发处理多个客户端的 TCP 连接请求**，能够提高程序的性能，Linux 下实现 I/O 多路复用的系统调用主要有`select()`、`poll()`和`epoll()`。

在学习`select()`、`poll()`和`epoll()`三个系统调用之前，**先了解一下常见的几种 I/O 模型**。

1.<font color = red>BIO（Blocking IO）模型</font>

我们知道，`read()`系统调用是阻塞的，服务器端会一直阻塞等待客户端发来的消息，这时，如果客户端一直不发消息，服务器端就一直阻塞，无法处理新的 TCP 连接请求。**BIO 模型通过多进程或多线程技术，将处理 TCP 连接请求的逻辑和 TCP 通信的逻辑分别用不同的进程或线程**，实现了在服务器端并发处理客户端的 TCP 连接请求。

<center>
  <img src = "./images/第五章 Linux网络编程/5-25 网络通信BlockingIO模型.png">
</center>


2.<font color = red>非阻塞，忙轮询 I/O 模型</font>

在该 I/O 模型下，如果程序需要从文件描述符对应的读缓冲区中读数据，**不管读缓冲区有没有数据，都会一直访问读缓冲区（忙轮询）**。

<center>
  <img src = "./images/第五章 Linux网络编程/5-25 网络通信非阻塞&忙轮询IO模型.png">
</center>


3.<font color = red>NIO(None Blocking IO) 模型</font>

NIO 模型和 BIO 模型本质的区别是，NIO 模型是非阻塞的，该模型虽然解决了**服务器端能够并发处理多个客户端的 TCP 连接请求**，但是由于非阻塞，且每次都会访问所有的缓冲区，会造成大量的系统资源浪费。

<center>
  <img src = "./images/第五章 Linux网络编程/5-25 网络通信NIO模型.png">
</center>
4.<font color = red>I/O 多路转接技术</font>

I/O 多路转接技术的基本原理是**委托内核监听文件描述符的缓冲区，只有文件描述符的缓冲区有 I/O 操作时，程序才会处理对应的缓冲区**，和 NIO 不同的是，I/O 多路转接技术是阻塞的，节省了不必要的系统资源浪费。

**第一种 I/O 多路转接技术**是 `select()` 和 `poll()` 系统调用实现的。

<center>
  <img src = "./images/第五章 Linux网络编程/5-25 select&poll多路转接技术.png">
</center>


**第二种 I/O 多路转接技术**是 `epoll()` 系统调用实现的。

<center>
  <img src = "./images/第五章 Linux网络编程/5-25 IO多路转接技术1.png">
</center>


### 5.20.1 select 多路复用

> 主旨思想：
>
> 1. 首先要构造一个关于文件描述符的列表，**将要监听的文件描述符添加到该列表中**。
> 2. 调用 `select()` 系统函数，**监听该列表中的文件描述符**，直到这些文件描述符对应缓冲区中的一个或者多个**进行 I/O 操作时**，该函数才返回。
>    - `select()` 系统调用是**阻塞的**。
>    - `select()` 系统调用**对文件描述符的检测操作是由内核完成的**。
> 3. 在 `select()` 系统调用返回时，它会告诉进程有多少（哪些）文件描述符对应的缓冲区**要进行 I/O 操作**。

<font color = green>`select()` 系统调用相关函数</font>

```c
// I/O 多路转接技术：select() 系统调用介绍
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

/*
    函数功能：监听多个文件描述符，直到这些文件描述符中有一个或者多个可以进行 I/O 操作时，该函数才退出阻塞状态，并且返回
    函数参数： sizeof(fd_set) = 128 bytes = 1024 bits, 可以存储 1024 个文件描述符缓冲区的状态，当文件描述符读缓冲区可以操作时，对应的 readfds 集合中的 bit 位值是 1
    - nfds：委托内核检测的最大文件描述符的值 + 1（文件描述符个数），因为文件描述符集合是用数组实现的，而数组索引下标为 n 对应的位置是 n + 1
    - readfds：要检测读缓冲区的文件描述符集合，一旦这些文件描述符集合的读缓冲区有数据，解除阻塞
    - writefds：要检测写缓冲区的文件描述符集合，一旦这些文件描述符集合的写缓冲区可以写数据，解除阻塞
    - exceptfds：要检测发生异常的文件描述符集合
    - timeout：设置 select() 函数阻塞的时间
        struct timeval {
            long    tv_sec;
            long    tv_usec;
        };
        - NULL：select() 函数永久阻塞，直到检测到了文件描述符有变化
        - tv_sec = 0 tv_usec = 0，不阻塞
        - tv_sec > 0 tv_usec > 0，阻塞对应的时间

    返回值
    - 成功：> 0，表示检测的 readfds、writefds 和 exceptfds 集合中，发生变化的文件描述符个数（一个文件描述符，对应一个 bit 位）
    - 失败：-1，并且设置错误号
*/
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

/*
    函数功能：将指定文件描述符，所在文件描述符集合的标志位置 0
    函数参数
    - fd：指定的文件描述符
    - set：指定的文件描述符集合
*/
void FD_CLR(int fd, fd_set *set);

/*
    函数功能：判断文件描述符，所在文件描述符集合对应位置，标志位的 bit 值
    函数参数
    - fd：指定的文件描述符
    - set：指定的文件描述符集合
    返回值：0 或 1，表示 bit 位上的值
*/
int  FD_ISSET(int fd, fd_set *set);

/*
    函数功能：将指定文件描述符，所在文件描述符集合的标志位置 1
    函数参数
    - fd：指定的文件描述符
    - set：指定的文件描述符集合
*/
void FD_SET(int fd, fd_set *set);

/*
    函数功能：将文件描述符集合 set 中的所有 bit 位，初始化为 0
    函数参数
    - set：需要初始化的文件描述符集合
*/
void FD_ZERO(fd_set *set);
```

<font color = green>使用 `select()` 实现 I/O 多路复用（转接）</font>

```c
#include<stdio.h>
#include<arpa/inet.h>
#include<string.h>
#include<stdlib.h>
#include<string.h>
#include<sys/select.h>
#include<unistd.h>

int main() {
    // 创建 socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(-1);
    }
    // 绑定
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    server_addr.sin_addr.s_addr = INADDR_ANY;       // 表示使用服务器端的任意 IP 地址，因为服务器可能有多网卡，或者 DHCP 服务导致 IP 地址动态变化
    int ret1 = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(-1);
    }
    // 监听
    int ret2 = listen(listen_fd, 128);  // 已完成三次握手的连接队列和未完成三次握手的未连接队列总和
    if (ret2 == -1) {
        perror("listen");
        exit(-1);
    }
    // 创建一个 fd_set 文件描述符集合，存放需要检测的文件描述符
    fd_set readfds, tmp;

    // 初始化 fd_set，并且设置指定文件描述符值为 1
    FD_ZERO(&readfds);
    FD_SET(listen_fd, &readfds);

    int max_fd = listen_fd;

    // 接受客户端发来的连接请求
    while (1) {
        /*
            tmp 变量防止 select() 函数覆盖 readfds 中的内容，因为客户端可能会停止发送数据，但是客户端并没有断开连接（这种情况内核会覆盖 tmp 集合中的标志位为 0），如果 readfds 传入 select() 函数，读缓冲区数据为空时，对应的 fd 标志位会被置 0
        */
        tmp = readfds;
        // select() 系统调用函数，让内核帮忙检测哪些文件描述符对应的读缓冲区有数据，阻塞函数
        int ret3 = select(max_fd + 1, &tmp, NULL, NULL, NULL);

        if (ret3 == -1) {
            perror("select");
            exit(-1);
        }

        else if (ret3 > 0) {
            // 有新的客户端连接进来，如果没有新的客户端连接，listen_fd 的读缓冲区没有内容，tmp[listen_fd]的值会被内核设置为 0
            if (FD_ISSET(listen_fd, &tmp)) {
                char client_ip[16] = { '\0' };
                unsigned short port = 0;
                struct sockaddr_in client_addr;
                socklen_t client_addr_len = (socklen_t)sizeof(client_addr);
                int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &client_addr_len);
                if (communication_fd == -1) {
                    perror("accept");
                    exit(-1);
                }

                // 获取客户端信息
                inet_ntop(AF_INET, (void*)&client_addr.sin_addr.s_addr, client_ip, (socklen_t)sizeof(client_ip));
                port = ntohs(client_addr.sin_port);
                printf("client ip = %s, port = %d\n", client_ip, port);

                // 将对应的通信文件描述符加入到文件描述符集合中
                FD_SET(communication_fd, &readfds);

                max_fd = max_fd > communication_fd ? max_fd : communication_fd;
            }

            // 通信
            // 从监听文件描述符后一位遍历，一般的，监听文件描述符的值最小
            for (int i = listen_fd + 1;i <= max_fd;++i) {
                if (FD_ISSET(i, &readfds)) {
                    // 存在文件描述符对应的 bit 标志位为 1，说明客户端发来了消息
                    char receive_buf[1024] = { '\0' };
                    int ret4 = read(i, receive_buf, sizeof(receive_buf));
                    if (ret4 == -1) {
                        perror("read");
                        exit(-1);
                    }
                    else if (ret4 == 0) {
                        printf("client closed\n");
                        close(i);
                        FD_CLR(i, &readfds);    // 完成通信，移除对应的文件描述符
                    }
                    else if (ret4 > 0) {
                        printf("receive data of client: %s\n", receive_buf);
                        write(i, receive_buf, sizeof(receive_buf));
                    }
                }
            }
        }

        else if (ret3 == 0) {
            // 读缓冲区文件描述符集合，不存在 bit 位变化，实际上这个分支不可能执行到，当 ret3 == 0 时，select() 会阻塞
            continue;
        }
    }

    // 关闭文件描述符
    close(listen_fd);

    return 0;
}
```

### 5.20.2 poll 多路复用

在上面编写的 `select()` I/O多路复用代码中，其实可以隐约的发现，`select()`I/O 多路复用技术是存在以下缺点的：

> - 每次调用 `select()`，都需要**把文件描述符集合从用户态拷贝到内核态**，文件描述符很多的时候，这个拷贝的开销会很大（<font color = green>拷贝文件描述符集合操作在`select()`系统调用内部实现</font>）。
> - 每次调用`select()`，都需要**在内核遍历传递进来的文件描述符集合**，同理，文件描述符很多的时候，这个遍历开销也会很大（<font color = green>遍历文件描述符集合操作，由我们自己编写的`for()`循环实现</font>）。
> - `select()`系统调用支持的文件描述符数量太小，默认是 1024。
> - 文件描述符集合不能重用，每次都需要重置（因为如果文件描述符对应缓冲区中没有内容，**内核会修改文件描述符对应标志位的值为 0**）。
>
> <center>
>   <img src = "./images/第五章 Linux网络编程/5-28 select的IO多路复用技术缺点.png">
> </center>
>**poll 多路复用是对 select 多路复用的一个改进，其中，改进的点如下：**
> 
>- poll 多路复用文件描述符集合的**大小可以自定义**。
> - poll 多路复用文件描述符集合**可以重用**。
> 
>综上，poll 多路复用解决了 select 多路复用的 3 和 4 两个缺点。

<font color = green>poll 多路复用系统调用相关函数</font>

```c
#include <poll.h>

struct pollfd {
  int   fd;           // 委托内核需要检测的文件描述符
  short events;       // 委托内核检测文件描述符的什么事件
  short revents;      // 文件描述符实际发生的事件
};

/*
    函数功能：实现 I/O 多路复用（服务器端可以并发处理 TCP 通信），对 select 的一个改进
    函数参数
    - fds：结构体数组指针，需要检测文件描述符的集合
    - nfds：第一个参数数组中，最后一个有效元素的下标 + 1，即表示委托内核需要遍历文件描述符边界
    - timeout：poll() 函数的阻塞时长
        - 0：不阻塞
        - (-1)：阻塞，当检测到需要检测的文件描述符有变化，解除阻塞
        - (>0)：阻塞时长，million second，毫秒
    返回值
    - 失败 ：-1
    - 成功：大于 0，返回检测到的集合中，有多少个文件描述符的 revents 值是非 0，等于 0，表示在 poll() 的阻塞时间内，没有文件描述符可以操作
*/
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

`struct pollfd` 结构体中，`events` 和 `revents` 的相关宏值如下。

|   事件   |     宏值     | 作为 `events` 的值 | 作为 `revents` 的值 |         说明         |
| :------: | :----------: | :----------------: | :-----------------: | :------------------: |
|  读事件  |   `POLLIN`   |         √          |          √          | 普通或优先带数据可读 |
|          | `POLLRDNORM` |         √          |          √          |     普通数据可读     |
|          | `POLLRDBAND` |         √          |          √          |   优先级带数据可读   |
|          |  `POLLPRI`   |         √          |          √          |   高优先级数据可读   |
|  写事件  |  `POLLOUT`   |         √          |          √          | 普通或优先带数据可写 |
|          | `POLLWRNORM` |         √          |          √          |     普通数据可写     |
|          | `POLLWRBAND` |         √          |          √          |   优先级带数据可写   |
| 错误事件 |  `POLLERR`   |                    |          √          |       发生错误       |
|          |  `POLLHUP`   |                    |          √          |       发生挂起       |
|          |  `POLLNVAL`  |                    |          √          |  描述不是打开的文件  |

<font color = green>使用 `poll()` 系统调用实现 I/O 多路复用（转接）</font>

```c
// poll IO 多路复用（转接）代码的编写
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/poll.h>

int main(int argc, char* argv[]) {
    // 监听socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 绑定 IP + port
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    server_addr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 监听
    ret = listen(listen_fd, 128);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 初始化监听的文件描述符集合
    struct pollfd fds[1024];
    int nfds = 0;
    for (int i = 0;i < 1024;++i) {
        fds[i].fd = -1;
        fds[i].events = POLLIN;     // 委托内核检测文件描述符读缓冲区内容
    }


    fds[0].fd = listen_fd;
    int fds_len = sizeof(fds) / sizeof(fds[0]);
    ++nfds;

    // 并发处理与客户端的 TCP 通信
    while (1) {
        // 委托内核检测文件描述符对应的缓冲区是否有内容
        int ret1 = poll(fds, nfds, -1);
        if (ret1 == 0) {
            // 没有可操作的文件描述符
            continue;
        }
        else if (ret1 == -1) {
            perror("poll");
            exit(-1);
        }
        else {
            if ((fds[0].revents & POLLIN) == POLLIN) {
                // 有新的客户端连接进来
                struct sockaddr_in client_addr;
                socklen_t addr_len = sizeof(client_addr);
                int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &addr_len);
                if (communication_fd == -1) {
                    perror("accept");
                    exit(-1);
                }
                printf("connect success.\n");

                unsigned short client_port = htons(client_addr.sin_port);
                char client_ip[16] = { '\0' };
                inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
                printf("client_ip = %s, client_port = %u.\n", client_ip, client_port);

                // 将 communication_fd 加入到文件描述符集合中
                for (int i = 1;i < fds_len;++i) {
                    if (fds[i].fd == -1) {
                        // 找到一个可用的文件描述符
                        fds[i].fd = communication_fd;
                        fds[i].events = POLLIN;
                        if (i > nfds - 1) {
                            // 更新最大文件描述符集合索引，这里有技巧性，因为可能中间索引的文件描述符先被释放
                            ++nfds;
                        }
                        // 找到一个可用的文件描述符，记得退出循环
                        break;
                    }
                }
            }

            // 通信
            for (int i = 1;i < nfds;++i) {
                if ((fds[i].revents & POLLIN == POLLIN) && (fds[i].fd != -1)) {
                    char receive_buf[1024] = { '\0' };
                    int ret2 = read(fds[i].fd, receive_buf, sizeof(receive_buf));
                    if (ret2 == -1) {
                        perror("read");
                        exit(-1);
                    }
                    else if (ret2 == 0) {
                        // 客户端连接断开
                        printf("client close\n");
                        close(fds[i].fd);
                        fds[i].fd = -1;
                        fds[i].events = POLLIN;
                    }
                    else if (ret2 > 0) {
                        printf("receive data: %s\n", receive_buf);
                        write(fds[i].fd, receive_buf, sizeof(receive_buf));
                    }
                }
            }
        }
    }

    // 关闭监听用的文件描述符
    close(listen_fd);

    return 0;
}
```

### 5.20.3 epoll 多路复用

> <center>
> <img src = "./images/第五章 Linux网络编程/5-29 epoll的IO多路复用技术.png">
> </center>

上图是 epoll 多路复用的基本工作流程，其中，`struct rb_root rbr;` 结构体存放了**委托内核需要帮忙检测的文件描述符集合**，`struct list_head rdlist;`  结构体存放了**内核检测到对应缓冲区内容发生变化的文件描述符集合**。

<font color = green>`epoll()` 系统调用相关函数</font>

```c
#include <sys/epoll.h>

struct eventepoll{
  // ...
  struct rb_root rbr;
  struct list_head rdlist;
  // ...
};

/*
    函数功能：在内核中创建一个新的 epoll 实例，该 epoll 实例中有两个比较重要的数据
    - 一个是需要检测的文件描述符集合（红黑树存储）
    - 一个是就绪列表，存放缓冲区数据发生改变的文件描述符集合（双向链表）
    函数参数：
    - size：没有实际意义，传递一个大于 0 的值即可
    返回值
    - 成功：返回大于零的值，是一个文件描述符
    - 失败：返回 -1 并且设置错误号
*/
int epoll_create(int size);

typedef union epoll_data {
  void        *ptr;
  int          fd;
  uint32_t     u32;
  uint64_t     u64;
} epoll_data_t;

struct epoll_event {
  uint32_t     events;	// 需要检测的事件
  epoll_data_t data;    // 文件描述符相关信息
};
/*
	常见的 epoll 检测事件：
    - EPOLLIN
    - EPOLLOUT
    - EPOLLERR
    - EPOLLET
*/

/*
    函数功能：对 epoll_create() 创建的实例进行管理，向文件描述符集合中，添加文件描述符信息、删除信息、修改信息
    函数参数
    - epfd：epoll 实例对应的文件描述符
    - op：对 epoll 实例存储的文件描述符集合进行什么样的操作
        - EPOLL_CTL_ADD：添加文件描述符
        - EPOLL_CTL_MOD：修改文件描述符
        - EPOLL_CTL_DEL：删除文件描述符
    - fd：要进行操作的文件描述符
    - event：绑定操作文件描述符所发生的事件
    返回值
    - 成功：0
    - 失败：-1，并且设置相关错误号
*/
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

/*
    函数功能：检测 epoll 实例的文件描述符集合中，有多少文件描述符对应的缓冲区发生了变化
    函数参数
    - epfd：epoll 实例对应的文件描述符
    - events：传出参数，保存发生了变化的文件描述符信息
    - maxevents：第二个参数 events 结构体数组的大小
    - timeout：阻塞时间
        - 0：不阻塞
        - -1：阻塞，直到检测到文件描述符数据发生变化，解除阻塞
        - >0：阻塞的时长（microsecond）
    返回值
    - 成功：返回缓冲区发生变化的文件描述符个数
    - 失败：返回 -1 并且设置错误号
*/
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

<font color = green>使用 `epoll()` 系统调用实现 I/O 多路复用</font>

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<arpa/inet.h>
#include<sys/epoll.h>

int main(int argc, char* agrv[]) {
    // 创建 socket
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("listen");
        exit(-1);
    }

    // 绑定 ip + port
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(9999);
    int ret1 = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(-1);
    }
    // 监听
    listen(listen_fd, 128);

    // 创建一个 epoll 实例
    int epoll_fd = epoll_create(100);
    if (epoll_fd == -1) {
        perror("epoll_fd");
        exit(-1);
    }

    // 将 listen_fd 加入到 epoll 实例中
    struct epoll_event listen_epev;    // 绑定需要检测的事件
    listen_epev.events = EPOLLIN;
    listen_epev.data.fd = listen_fd;
    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, listen_fd, &listen_epev);

    // 存储检测到发生变化的文件描述符
    struct epoll_event epevs[1024];
    while (1) {
        int ret2 = epoll_wait(epoll_fd, epevs, 1024, -1);
        if (ret2 == -1) {
            perror("epoll_wait");
            exit(-1);
        }
        printf("ret2 = %d\n", ret2);

        for (int i = 0;i < ret2;++i) {
            if (epevs[i].data.fd == listen_fd) {
                // 有新的客户端连接进来
                struct sockaddr_in client_addr;
                socklen_t addr_len = sizeof(client_addr);
                int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &addr_len);
                if (communication_fd == -1) {
                    perror("accept");
                    exit(-1);
                }
                printf("connect success.\n");

                char client_ip[16] = { '\0' };
                unsigned short client_port = ntohs(client_addr.sin_port);
                inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
                printf("client ip = %s, client_port = %u.\n", client_ip, client_port);

                // 将新客户端的通信文件描述符加入到 epoll 实例中
                struct epoll_event communication_epev;
                communication_epev.data.fd = communication_fd;
                communication_epev.events = EPOLLIN;
                epoll_ctl(epoll_fd, EPOLL_CTL_ADD, communication_fd, &communication_epev);
            }
            else {
                // 没有新的客户端连接，进行通信，检测读事件
                if (epevs[i].events & EPOLLIN != EPOLLIN) {
                    continue;
                }
                char receive_buf[1024] = { '\0' };
                int ret3 = read(epevs[i].data.fd, receive_buf, sizeof(receive_buf));
                if (ret3 == -1) {
                    perror("read");
                    exit(-1);
                }
                else if (ret3 == 0) {
                    printf("client closed.\n");
                    // 删除 epoll 实例里面的文件描述符信息
                    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, epevs[i].data.fd, NULL);
                    close(epevs[i].data.fd);
                }
                else {
                    printf("receive data: %s\n", receive_buf);
                    write(epevs[i].data.fd, receive_buf, sizeof(receive_buf));
                }
            }
        }
    }

    // 关闭相应的文件描述符
    close(epoll_fd);
    close(listen_fd);
  
    return 0;
}
```

<font color = green>epoll 的工作模式</font>

> **LT 模式（水平触发，epoll 实例的默认工作模式）：**LT（Level Triggered）是缺省的工作方式，并且同时支持 block 和 no block 的 socket。在这种模式下，**内核告诉你一个文件描述符是否就绪了**，然后可以对这个**就绪的文件描述符进行 IO 操作**。如果不做任何操作，**内核还是会继续通知你**。
>
> 假设委托内核检测读事件 ==> 检测文件描述符的读缓冲区
>
> - 读缓冲区中有数据 ==> epoll 实例检测到了会给用户通知
>   - 用户不读数据，数据一直在缓冲区，**epoll 实例会一直通知**
>   - 用户只读了一部分数据，**epoll 实例还会通知**
>   - 缓冲区数据读完了，不通知
>
> 
>
> **ET 模式（边沿触发，需要手动设置）：**ET（Edge Triggered）是高速工作方式，只支持 no block 的 socket。在这种模式下，当文件描述符从未就绪变为就绪时，内核通过 epoll 告诉你。然后内核会假设你直到文件描述符已经就绪，并不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了。但是请注意，如果一直不对这个文件描述符作 IO 操作（从而导致它再次变为未就绪），内核不会发送更多的通知（only once）。
>
> ET 模式在很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。epoll 工作在 ET 模式的时候，必须使用非阻塞的套接字，以避免由于一个文件句柄的 阻塞读/阻塞写 操作，把处理多个文件描述符的任务饿死。
>
> 假设委托内核检测读事件 ==> 检测文件描述符的读缓冲区
>
> - 读缓冲区中有数据 ==> epoll 实例检测到了会给用户通知
>   - 用户不读数据，数据一直在缓冲区，**epoll 实例下次检测的时候就不通知了**
>   - 用户只读了一部分数据，**epoll 实例不通知**
>   - 缓冲区数据读完了，不通知

## 5.21 UDP 通信

<center>
  <img src = "./images/第五章 Linux网络编程/5-32 UDP 通信.png" width = "70%">
</center>


### 5.21.1  UDP 通信相关函数

```c
#include <sys/types.h>
#include <sys/socket.h>

/*
    函数功能：向网络上的其它 socket 发送数据（可面向无连接）
    函数参数
    - sockfd：通信用的文件描述符
    - buf：要发送数据的字符串数组
    - len：发送数据的长度
    - flags：默认使用 0 即可
    - dest_addr：网络通信另外一端的地址信息（目的端的 socket 信息）
    - addrlen：参数 dest_addr 所占内存的大小
    返回值
    - 成功：返回发送数据的字节数
    - 失败：返回 -1 并且设置错误号
*/
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,const struct sockaddr *dest_addr, socklen_t addrlen);

/*
    函数功能：接收网络上其它 socket 发送过来的数据（可面向无连接）
    函数参数
    - sockfd：通信用的文件描述符
    - buf：接收数据的字符串数组
    - len：数组的大小
    - flags：默认使用 0 即可
    - src_addr：网络通信另外一端的地址信息（源端的 socket 信息）
    - addrlen：参数 src_addr 所占内存的大小
    返回值
    - 成功：返回接收数据的字节数
    - 失败：返回 -1 并且设置错误号
*/
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```

### 5.21.2 UDP 通信使用案例

<font color = green> UDP 通信服务器端实现 </font>

> UDP 服务器端通信流程：
>
> - 创建通信用的 socket
> - 将通信用的 socket 绑定服务器端的 IP 和 port
> - 使用`sendto()`系统调用发送数据，`recvfrom()`系统调用接收数据
> - 关闭通信用的文件描述符

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h>

int main(int argc, char* argv[]) {
    // 1. 创建 UDP 通信用的 socket , PF_INET 的使用更加关注协议族，AF_INET 的使用更加关注地址族
    int communication_fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (communication_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2. 绑定 ip + port
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;   // 接收所有网络接口的 UDP 数据报
    server_addr.sin_port = htons(9999);
    int ret1 = bind(communication_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(-1);
    }

    // 3. 进行通信
    while (1) {
        // 接收客户端发来的 UDP 数据报
        char receive_buf[128] = { '\0' };
        struct sockaddr_in client_addr;
        socklen_t addr_len = sizeof(client_addr);
        char client_ip[16] = { '\0' };
        unsigned short client_port = 0;

        int ret2 = recvfrom(communication_fd, receive_buf, sizeof(receive_buf), 0, (struct sockaddr*)&client_addr, &addr_len);
        if (ret2 == -1) {
            perror("recvfrom");
            exit(-1);
        }
        else if (ret2 == 0) {
            printf("client closed.\n");
            break;
        }

        inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
        ntohs(client_addr.sin_port);

        // 输出客户端信息 + 接收的数据
        printf("client ip = %s, port = %u is sending data: %s.\n", client_ip, client_port, receive_buf);

        // 向客户端发送数据
        int ret3 = sendto(communication_fd, receive_buf, sizeof(receive_buf), 0, (struct sockaddr*)&client_addr, addr_len);
        if (ret3 == -1) {
            perror("sendto");
            exit(-1);
        }

    }

    // 4. 关闭通信用的文件描述符
    close(communication_fd);

    return 0;
}
```

<font color = green> UDP 通信客户端实现 </font>

> UDP 客户端通信流程：
>
> - 创建通信用的 socket
> - 指定服务器端的 IP 和 port
> - 使用`sendto()`系统调用发送数据，`recvfrom()`系统调用接收数据
> - 关闭通信用的文件描述符

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h>

int main(int argc, char* argv[]) {
    // 1. 创建 UDP 通信用的 socket
    int communication_fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (communication_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2. 指定数据接收方的 socket (这里是服务器)
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    server_addr.sin_port = htons(9999);

    // 3. 通信
    int count = 1;
    while (1) {
        sleep(1);

        char send_buf[128] = { '\0' };
        char receive_buf[128] = { '\0' };
        sprintf(send_buf, "I am client, count = %d", count++);
        // 发送数据
        int ret1 = sendto(communication_fd, send_buf, sizeof(send_buf), 0, (struct sockaddr*)&server_addr, sizeof(server_addr));
        if (ret1 == -1) {
            perror("sendto");
            exit(-1);
        }

        // 接收数据
        int ret2 = recvfrom(communication_fd, receive_buf, sizeof(receive_buf), 0, NULL, NULL);
        if (ret2 == -1) {
            perror("recvfrom");
            exit(-1);
        }
        else if (ret2 == 0) {
            printf("server closed.\n");
            break;
        }

        printf("receive server data: %s\n", receive_buf);
    }

    // 4. 关闭 UDP 通信用的文件描述符
    close(communication_fd);

    return 0;
}
```

## 5.22 广播

向子网中多台计算机发送消息，并且子网中所有计算机都可以接收到发送方发送的消息，**每一个广播消息都包含一个特殊的 IP 地址，这个 IP 地址中，主机号部分全为（主机号全为 0 代表本网络）**。

- 广播只能在**局域网**中使用。
- 客户端需要**绑定服务器端广播使用的端口**，才可以接收到广播消息。

<center>
  <img src = "./images/第五章 Linux网络编程/5-33 广播1.png" width = "60%">
</center>


### 5.22.1 使用`setsockopt()`设置广播

```c
#include <sys/types.h>         
#include <sys/socket.h>

/*
	函数功能：设置 socket 的相关选项，这个系统调用允许我们修改 socket 的行为，例如调整 socket 对应接收缓冲区的大小、启用或禁用某些协议特性等。
	函数参数
	- sockfd：需要设置的 socket 对应的文件描述符
	- level: 指定设置选项所在的协议层，常见包括：
		- SOL_SOCKET: 表示设置选项是套接字层面的
		- IPPROTO_TCP 或 IPPROTO_IP: 针对 TCP 或 IP 层面的选项
		- IPPROTO_IPV6: 针对 IPv6 协议的选项
  - optname：指定具体要设置 socket 的哪一个选项，比如在 SOL_SOCKET 下，有 SO_REUSEADDR 和 SO_KEEPALIVE 等
  - optval：指向包含 socket 选项值的地址
  - optlen：optval 缓冲区的长度
	返回值
	- 成功：返回 0
	- 失败： 返回 -1 并且设置错误号
	
	也可以通过该系统调用设置端口复用 
*/
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
/*
	使用该系统调用设置广播
	- sockfd：需要设置广播的 socket 对应的套接字
	- level：SOL_SOCKET
	- optname：SO_BROADCAST
	- *optval：赋值为1
*/
```

### 5.22.2 广播使用案例

<font color = green>服务器端实现 UDP 通信广播</font>

> UDP 通信广播服务器端（发送数据）实现流程
>
> - 创建通信用的 socket
> - 设置 socket 允许广播属性
> - 设置广播数据接收的 IP 和 port
> - 发送广播数据
> - 关闭通信用的文件描述符

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h>

int main(int argc, char* argv[]) {
    // 1. 创建 UDP 通信用的 socket , PF_INET 的使用更加关注协议族，AF_INET 的使用更加关注地址族
    int communication_fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (communication_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2. 设置通信的 socket 广播属性
    int optval = 1;     // 允许广播
    setsockopt(communication_fd, SOL_SOCKET, SO_BROADCAST, &optval, sizeof(optval));

    // 3. 设置广播的 IP + port
    struct sockaddr_in client_addr;
    client_addr.sin_family = AF_INET;
    inet_pton(AF_INET, "172.30.207.255", &client_addr.sin_addr.s_addr);
    client_addr.sin_port = htons(9999);

    socklen_t addr_len = sizeof(client_addr);

    // 4. 进行通信，发送广播消息
    int num = 0;
    while (1) {
        sleep(1);
        char send_buf[128] = { '\0' };
        sprintf(send_buf, "I am server, num = %d.", ++num);
        // 向客户端发送数据
        int ret3 = sendto(communication_fd, send_buf, sizeof(send_buf), 0, (struct sockaddr*)&client_addr, addr_len);
        if (ret3 == -1) {
            perror("sendto");
            exit(-1);
        }
        printf("%s\n", send_buf);
    }

    // 5. 关闭通信用的文件描述符
    close(communication_fd);

    return 0;
}
```

<font color = green>客户端实现 UDP 通信广播</font>

> UDP 通信广播客户端（接收广播数据）实现流程
>
> - 创建通信用的 socket
> - 将 socket 绑定客户端的 IP 和 port（指定接收哪个网卡，哪个端口的数据）
> - 接收客户端发送的广播数据
> - 关闭通信用的文件描述符

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h>

// 客户端接收服务器广播发来的消息
int main(int argc, char* argv[]) {
    // 1. 创建 UDP 通信用的 socket
    int communication_fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (communication_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2. 客户端绑定所有网络接口的 IP 和 port
    struct sockaddr_in client_addr;
    client_addr.sin_family = AF_INET;
    client_addr.sin_port = htons(9999);
    client_addr.sin_addr.s_addr = INADDR_ANY;
    int ret1 = bind(communication_fd, (struct sockaddr*)&client_addr, sizeof(client_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(-1);
    }

    // 3. 接收客户端发来的广播消息
    while (1) {
        char receive_buf[128] = { '\0' };
        // 接收数据，默认和 read 一样，是阻塞的
        int ret2 = recvfrom(communication_fd, receive_buf, sizeof(receive_buf), 0, NULL, NULL);
        if (ret2 == -1) {
            perror("recvfrom");
            exit(-1);
        }
        else if (ret2 == 0) {
            printf("server closed.\n");
            break;
        }
        printf("receive server data: %s\n", receive_buf);
    }

    // 4. 关闭 UDP 通信用的文件描述符
    close(communication_fd);

    return 0;
}
```

## 5.23 组播（多播）

单播地址标识**单个 IP 接口**，广播地址标识**某个子网的所有 IP 接口**，多播地址标识**一组 IP 接口**。单播和广播是寻址方案的**两个极端（要么单个要么全部）**，多播则是在两者之间提供的一种折中方案。多播数据报**只由对它感兴趣的接口接收**，由运行**相应多播会话应用系统（直播客户端利用了多播会话应用系统）**的主机上的接口接收。另外，广播一般局限于局域网内使用，而多播**既可以用于局域网，也可以跨广域网使用**。

> - 客户端**需要加入多播组**，才能接收到多播的数据
> - 多播可用于局域网，也可用于广域网

<center>
  <img src = "./images/第五章 Linux网络编程/5-34 组播（多播）1.png" width = "70%">
</center>


### 5.23.1 组播地址

> IP 多播通信必须依赖于 IP 多播地址，在 IPv4 中，IP 多播地址的范围从 `224.0.0.0` 到 `239.255.255.255`，并且被划分为**局部链接多播地址、预留多播地址和管理权限多播地址三类**：
>
> | IP 地址                     | 说明                                                         |
> | --------------------------- | ------------------------------------------------------------ |
> | 224.0.0.0 ~ 224.0.0.255     | 局部链接多播地址：是为路由协议和其它用途保留的地址，路由器并不转发属于此范围的 IP 数据报 |
> | 224.0.1.0 ~ 224.0.1.255     | 预留多播地址：公用组播地址，可用于 Internet，使用前需要申请  |
> | 224.0.2.0 ~ 238.255.255.255 | 预留多播地址：用户可用组播地址（临时组地址），全网范围内有效 |
> | 239.0.0.0 ~ 239.255.255.255 | 本地管理多播地址：可供组织内部使用，类似于私有 IP 地址，不能用于 Internet，可限制多播范围 |

### 5.23.2 使用`setsockopt()`设置组播

```c
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
/*
	服务器使用该系统调用设置多播的信息，外出接口
	- sockfd：需要设置组播的 socket 对应的文件描述符
	- level：IPPROTO_IP
	- optname：IP_MULTICAST_IF
	- *optval：struct in_addr
	
	客户端使用该系统调用加入到多播组
	- sockfd：需要加入到多播组的 socket 对应的文件描述符
	- level：IPPROTO_IP
	- optname：IP_ADD_MEMBERSHIP
	- *optval：struct ip_mreq
*/
struct ip_mreq{
    struct in_addr imr_multiaddr;  // 加入多播的 IP 地址
    struct in_addr imr_interface;	 // 本机的 IP 地址
};

typedef uint32_t in_addr_t;
struct in_addr{
	in_addr_t s_addr; 
};

```

## 5.24 本地套接字通信

本地套接字的作用：**本地的进程间通信**。

> - 有关系的进程间通信
> - 没有关系的进程间通信
>
> 本地套接字实现流程和网络套接字类似，一般采用 TCP 的通信流程。
>
> <center>
>   <img src = "./images/第五章 Linux网络编程/5-35 本地套接字通信.png" width = "90%">
> </center>
>
>
> ```c
> #include<sys/un.h>
> #define UNIX_PATH_MAX 108
> 
> struct sockaddr_un{
>   sa_family_t sun_family;				// 地址族协议 AF_LOCAL
>   char sun_path[UNIX_PATH_MAX];	// 套接字文件的路径，这是一个伪文件，大小永远是
> };
> ```

### 5.24.1 本地套接字通信流程

> 服务器端：
>
> - 创建监听用的 socket
> - 监听的套接字绑定本地的套接字文件（`*.sock`） ==> server 端
> - 监听
> - 等待接受连接请求
> - 通信
> - 关闭文件描述符
>
> 客户端：
>
> - 创建通信用的 socket
> - 通信的 socket 绑定本地的套接字文件（`*.sock`） ==> client 端
> - 连接服务器
> - 通信
> - 关闭文件描述符

### 5.24.2 本地套接字通信案例

<font color = green>本地套接字服务器端实现</font>

```c
/*
    本地套接字通信流程（服务器端）
*/
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<arpa/inet.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<string.h>
#include<sys/un.h>

int main(int argc, char* argv[]) {
    // 1. 创建监听用的 socket
    int listen_fd = socket(PF_LOCAL, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
    }

    // 2. 将 socket 绑定一个虚拟文件（相当于网络间通信绑定服务器端的网络接口）
    struct sockaddr_un server_addr;
    server_addr.sun_family = AF_LOCAL;
    strcpy(server_addr.sun_path, "server.sock");

    // 绑定虚拟磁盘文件之前，删除已经存在的虚拟磁盘文件
    int ret4 = unlink("./server.sock");
    if (ret4 == -1) {
        printf("not exist server.sock.\n");
    }

    int ret1 = bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("bind");
        exit(-1);
    }

    // 3. 监听
    int ret2 = listen(listen_fd, 128);
    if (ret2 == -1) {
        perror("listen");
        exit(-1);
    }

    // 4. 接受本地客户端发送的连接请求
    struct sockaddr_un client_addr;
    socklen_t addr_len = sizeof(client_addr);
    int communication_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &addr_len);
    if (communication_fd == -1) {
        perror("accept");
        exit(-1);
    }
    printf("client connect success, sun_path = %s.\n", client_addr.sun_path);

    // 5. 通信
    while (1) {
        char recv_buf[128] = { '\0' };
        int ret3 = read(communication_fd, recv_buf, sizeof(recv_buf));
        if (ret3 == -1) {
            perror("read");
            exit(-1);
        }
        else if (ret3 == 0) {
            printf("client closed.\n");
            break;
        }
        else {
            printf("receive client data: %s\n", recv_buf);
            write(communication_fd, recv_buf, sizeof(recv_buf));
        }
    }

    // 6. 关闭文件描述符
    close(communication_fd);
    close(listen_fd);
    return 0;
}
```

<font color = green>本地套接字客户端实现</font>

```c
/*
    本地套接字通信流程（客户端）
*/
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<arpa/inet.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<sys/un.h>
#include<string.h>

int main(int argc, char* argv[]) {
    // 1. 创建通信用的 socket
    int communication_fd = socket(PF_LOCAL, SOCK_STREAM, 0);
    if (communication_fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2. 通信的 socket 绑定本地的虚拟文件
    struct sockaddr_un client_addr;
    client_addr.sun_family = AF_LOCAL;
    strcpy(client_addr.sun_path, "client_sock");

    // 绑定虚拟磁盘文件之前，删除已经存在的虚拟磁盘文件
    int ret4 = unlink("./client.sock");
    if (ret4 == -1) {
        printf("not exist client.sock.\n");
    }
    int ret3 = bind(communication_fd, (struct sockaddr*)&client_addr, sizeof(client_addr));
    if (ret3 == -1) {
        perror("bind");
        exit(-1);
    }

    // 3. 连接本地套接字（服务器端）
    struct sockaddr_un server_addr;
    server_addr.sun_family = AF_LOCAL;
    strcpy(server_addr.sun_path, "server.sock");
    int ret1 = connect(communication_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret1 == -1) {
        perror("connect");
        exit(-1);
    }

    // 4. 通信
    int count = 0;
    while (1) {
        sleep(1);
        char send_buf[128] = { '\0' };
        char recv_buf[128] = { '\0' };
        sprintf(send_buf, "I am client, count = %d.", count++);
        write(communication_fd, send_buf, sizeof(send_buf));
        int ret2 = read(communication_fd, recv_buf, sizeof(recv_buf));
        if (ret2 == -1) {
            perror("read");
            exit(-1);
        }
        else if (ret2 == 0) {
            printf("server closed.\n");
            break;
        }
        else {
            printf("receive server data: %s\n", recv_buf);
        }
    }

    // 5. 关闭文件描述符
    close(communication_fd);

    return 0;
}
```

## 5.25 一些零碎知识点

- `ps aux | grep sshd` 命令解析

```bash
# ps 用于显示系统中当前运行的进程信息
# a 选项表示显示所有用户的进程
# u 选项表示以用户友好的格式显示进程信息
# x 选项表示没有控制终端的进程
# grep 命令用于在终端输出中检索包含指定字符串的行
# sshd 即 grep 要检索的指定字符串
```

- 查看网络信息相关的命令：`netstat`

```bash
# -a 参数，所有的socket
# -p 参数，显示正在使用 socket 的程序的名称
# -n 参数，直接使用 IP 地址，而不通过 DNS
```

