# Application Layer

## Principles of Network Applications

进行通信实际上是*进程* 而不是程序。

Socket（套接字）是应用程序和网络之间的应用程序编程接口（API).

![](http://img.070077.xyz/202203101458159.png)

- TCP service

***reliable** transport*: between sending and receiving process

*flow control:* sender won’t overwhelm receiver 

*congestion control:* throttle sender when network overloaded

***connection**-**oriented**:* setup required between client and server processes

does not provide: timing, minimum throughput guarantee, security

- UDP service:

***unreliable** data transfer* between sending and receiving process

does not provide: reliability, flow control, congestion control, timing, throughput guarantee, security, or connection setup.

## Web Http

### Non-persistent HTTP

RTT（round-trip time） : time for a small packet to travel from client to server and back![total response time is two RTTs plus the transmission time at the server of the HTML file.](http://img.070077.xyz/202203101656715.png)

Persistent（HTTP1.1）:  introduced multiple, pipelined GETs over single TCP connection. Subsequent requests and responses between the same client and server can be sent over the **same** connection.

> HTTP 通信时，除客户端和服务器以外，还有一些用于通信数据转发的应用程序配合服务器工作。
>
> - 代理：接收由客户端发送的请求并转发给服务器，接收服务器返回的响应并转发给客户端。
> - 网关：转发其他服务器通信数据的服务器，就像自己拥有资源的源服务器一样处理请求。
> - 隧道：是在客户端和服务器两者之间进行中转，并保持双方通信连接。

### HTTP Method

| Method | Usage                                                        |
| ------ | ------------------------------------------------------------ |
| GET    | user input sent from client to server in entity body of POST request message |
| POST   | include user data in URL field of HTTP GET request message (following a ‘?’) |
| HEAD   | requests headers (only) that would be returned *if specified* URL were requested with an HTTP GET method. |
| PUT    | uploads new file (object) to server；completely replaces file that exists at specified URL with content in entity body of POST HTTP request message |
| DELETE | allows a user, or an application, to delete an object on a Web server. |

### Cookies

![](http://img.070077.xyz/202203101659670.png)

Cookies can be used to: 

- track user behavior on a given website (first party cookies)

- track user behavior across multiple websites (third party cookies,第三方cookies) without user ever choosing to visit tracker site.
- eg. Referer to ad host.

### Web caches 

HTTP GET/response interaction is *stateless*：

- HTTP server does not remember anything about what happened during earlier steps in interacting with this HTTP client.

![Use Conditional GET - If-modified-since: <date>](http://img.070077.xyz/202203101702288.png)

Web Cache is also called *proxy server*.

### HTTP/2

*HTTP/2:*increased flexibility at *server* in sending objects to client:

- transmission order of requested objects based on client-specified object priority.
- divide objects into frames, schedule frames to mitigate *HOL*(Head Of Line) blocking

> head-of-line (HOL) blocking: small object may have to wait for transmission behind large object(s)
>
> *HTTP1.1:* introduced multiple, pipelined GETs over single TCP connection
>
> HTTP2: transmission order of requested objects based on client-specified object priority (not necessarily FCFS)

## E-mail

- mail servers

  - *mailbox* contains incoming messages for user

  - *message queue* of outgoing (to be sent) mail message

![](http://img.070077.xyz/202203110043765.png)

SMTP comparison with HTTP:

- HTTP: client pull  |  SMTP: client **push**

- both have ASCII command/response interaction, status codes

- HTTP: each object encapsulated in its own response message

- SMTP: multiple objects sent in multipart message

- SMTP uses **persistent** connections

- SMTP requires message (header & body) to be in 7-bit ASCII

- SMTP server uses CRLF.CRLF to determine end of message

## DNS

### How DNS works?

TLD：Top-level domain.

|         Interaction of the various DNS servers | Recursive queries in DNS（with DNS cache）     |
| ---------------------------------------------: | ---------------------------------------------- |
| ![](http://img.070077.xyz/202203110155710.png) | ![](http://img.070077.xyz/202203110156013.png) |

the local DNS server may be on the same LAN as the host; for a residential ISP, it is typically separated from the host by no more than a few routers. 

### DNS records

`(Name, Value, Type, TTL)`. TTL is the time to live of the resource record.

| Type  | Description                                                  |
| ----- | ------------------------------------------------------------ |
| A     | name is hostname ; value is IP address                       |
| NS    | name is domain ; value is hostname of authoritative name server for this domain |
| CNAME | name is alias name for some “canonical” (the real) name; value is canonical name |
| MX    | value is name of SMTP mail server associated with name       |

![](http://img.070077.xyz/202203110203493.png)

> DNS使用什么协议进行传输？
>
> 进行区域传送时使用TCP；域名解析时一般使用UDP协议，负载低、响应快、包体小（部分DNS也可以使用TCP，DNS同时占用UDP和TCP端口53）

## P2P

![P2P architecture can be self-scaling](http://img.070077.xyz/202203110340088.png)

Each torrent has an infrastructure node called a *tracker*. In deciding which chunks to request, requesting chunks uses a technique called **rarest first**.

Sending chunks: tit-for-tat, randomly select another peer, starts sending chunks every 30 secs.

## Video Streaming

Streaming video = encoding + DASH + playout buffering

DASH：Dynamic, Adaptive Streaming over HTTP

CDN：内容分发网。store/serve multiple copies of videos at multiple geographically distributed sites.

![](http://img.070077.xyz/202203110406851.png)



## HTTPS = HTTP+加密+认证+完整性保护

![](http://img.070077.xyz/202203180204837.png)

TCP/IP是可能被窃听的网络。HTTP 协议中没有加密机制，加密处理防止被窃听。加密的对象有：

- 将通信加密。可以通过和 SSL（Secure Socket Layer，安全套接层）或 TLS（Transport Layer Security，安全层传输协议）的组合使用，加密 HTTP 的通信内容。

- 将通信内容本身加密。把 HTTP 报文里所含的内容进行加密处理。

  ![根据密文和公钥，解密原文是技术不支持的](http://img.070077.xyz/202203180206343.png)

HTTP无法验证通信方的身份，可能遭遇伪装。SSL 还提供了*证书*，由第三方权威机构CA颁发。

![](http://img.070077.xyz/202203180203419.png)

HTTP 协议无法证明通信的报文完整性，常用的是 MD5 和 SHA-1 等散列值校验是否篡改。

### 加密技术

![安全通信机制](http://img.070077.xyz/202203180153787.png)

![](http://img.070077.xyz/202203180154860.png)

### 身份认证

HTTP/1.1 使用的认证方式如下：

- BASIC 认证（基本认证，extends HTTP/1

  其采用的 Base64 编码方式，不是加密处理，可直接解码；注销不灵活。

  ![](http://img.070077.xyz/202203180211063.png)

-  DIGEST 认证（摘要认证， from HTTP/1.1）

  不存在防止用户伪装的保护机制，不灵活。

  ![](http://img.070077.xyz/202203180213440.png)

-  SSL 客户端认证（要钱）

  双因素认证：认证过程中不仅需要密码这一个因素，还需要申请认证者提供其他持有信息。

  需要事先将客户端证书分发给客户端，且客户端必须安装此证书领取证书内客户端的公钥。

-  FormBase 认证（基于表单认证，主流）

  一般会使用 Cookie 来管理Session（会话）。

## Socket Programming

![](http://img.070077.xyz/202203110419219.png)

![](http://img.070077.xyz/202203110424271.png)

![](http://img.070077.xyz/202203110958344.png)



# Transport Layer

## Overview

A transport-layer protocol provides for logical communication between **application processes** running on different *hosts*, an application’s perspective. (Different  network layer: logical communication between *hosts*)

Two principal Internet transport protocols:

- **TCP:** Transmission Control Protocol
  - reliable, in-order delivery
  - congestion control 
  - flow control
  - connection setup

- **UDP:** User Datagram Protocol
  - unreliable, unordered delivery
  - no-frills extension of “best-effort” IP

- services not available: 
  - delay guarantees
  - bandwidth guarantees

## Multiplexing and Demultiplexing

Multiplexing, demultiplexing: based on segment, datagram header field values

![when creating socket, must specify host-local port](http://img.070077.xyz/202203130103869.png)

### For Connectionless

a UDP socket is fully identified by a two-tuple consisting of a destination IP address and a destination port number.

IP/UDP datagrams with *same* dest port #, but different source IP addresses and/or source port numbers will be directed to *same socket* at receiving host.

### For Connection-Oriented

a TCP socket is identified by a four-tuple: (source IP address, source port number, destination IP address, destination port number).

each socket associated with a different connecting client

![Welcome socket first](http://img.070077.xyz/202203130108889.png)

## UDP

UDP use: streaming multimedia apps (loss tolerant, rate sensitive)，DNS，SNMP，HTTP/3

### Why UDP？

- **Finer** application-level control over what data is sent, and when.
- No connection establishment.
- No connection state.
- Small packet header overhead.

### Segment

![报文段结构](http://img.070077.xyz/202203151434035.png)

### Checksum

![a celebrated end-end principle](http://img.070077.xyz/202203151435094.png)

At the receiver, all four 16-bit words are added, including the checksum. If no errors are introduced into the packet, then clearly the sum at the receiver will be *1111111111111111*. If one of the bits is a 0, then we know that errors have been introduced into the packet.

## RDT

![Sender can`t see Receiver directly](http://img.070077.xyz/202203152141739.png)

### RDT2: channel with Bit Errors

![Stop and wait](http://img.070077.xyz/202203152142374.png)

what happens if ACK/NAK corrupted? 

retransmit？ receiver cannot know - an arriving packet contains new data or is a retransmission.

RDT2.1 Data with sequence number.

![state must “remember” whether “expected” pkt should have seq # of 0 or 1](http://img.070077.xyz/202203152150704.png)

![When an out-of-order packet is received, the receiver sends a positive acknowledgment for the packet it has received.](http://img.070077.xyz/202203152149105.png)

RDT2.2: use ACKs only.

duplicate ACK at sender results in same action as NAK.

![NAK_FREE](http://img.070077.xyz/202203152153435.png)

### RDT3：channel with errors and loss

Implementing a time-based retransmission mechanism requires a countdown **timer** that can interrupt the sender after a given amount of time has expired. 

![alter-nating-bit protocol](http://img.070077.xyz/202203152157852.png)

这依旧是一个 stop-and-wait 协议，带宽利用率（性能）不高。

### Pipelined Reliable Data Transfer Protocols

![3-packet pipelining increases utilization by a factor of 3](http://img.070077.xyz/202203152201169.png)

Two basic approaches toward pipelined error recovery can be identified: Go-Back-N and selective repeat.

- Go - Back - N(GBN)

![Sliding-window protocol](http://img.070077.xyz/202203152204889.png)

![discard: 接收方便](http://img.070077.xyz/202203152206759.png)

- Selective Repeat (SR)

![发送前重新确认是必要的](http://img.070077.xyz/202203152209545.png)

如果ACK未被发送方接收，则须重传将发送方窗口向前移动。

这也是难度所在 ：发送方和接收方**窗口不完全一致**。

![buffer for backup](http://img.070077.xyz/202203152211824.png)

Why checked the left ACK?the sender may not have received an ACK for that packet yet.

---

![](http://img.070077.xyz/202203152213144.png)

---

## TCP

### Why TCP?

- reliable, in-order byte **stream**: no “message boundaries"

- A TCP connection provides a full-duplex（全双工，即双向数据传输） service.
- pipelining: TCP congestion and flow control set window size
- connection-oriented: handshaking (exchange of control messages) initializes sender, receiver **state** before data exchange
- flow controlled: sender will not overwhelm receiver

![TCP segments are passed down to the network layer](http://img.070077.xyz/202203160725337.png)

### Segment

![Structure](http://img.070077.xyz/202203160740685.png)

The sequence number for a segment is therefore the byte-**stream** number of the **first** byte in the segment.

The ACK number that Receiver puts in its segment is the sequence number of the **next** byte expecting from Sender.

![](http://img.070077.xyz/202203160745352.png)

TCP only acknowledges bytes up to the **first** missing byte in the stream, TCP is said to provide *cumulative acknowledgments*.

### RTT

The connection’s *round-trip time* (RTT)  is the time from when a segment is sent until it is acknowledged.

TCP通过EWMA（指数加权移动平均. exponential weighted moving average ）维护一个RTT均值。建议取值$\alpha = 0.125$

$ EstimatedRTT = (1 – \alpha)\times EstimatedRTT + \alpha\times SampleRTT $

同时维护DevRTT（RTT偏差），即当前SampleRTT与EstimatedRTT的差值。一般取$\beta = 0.25$

$$ DevRTT = (1 – β)  DevRTT + β  | SampleRTT – EstimatedRTT | $$

于是得到超时间隔：

$ TimeoutInterval = EstimatedRTT + 4 \times DevRTT $

### Reliable Data Transfer

面向事件的处理方案。以下：

![](http://img.070077.xyz/202203160757738.png)

e,g. scenario 

![use updated Timeinterval](http://img.070077.xyz/202203160756318.png)

- Fast retransmit（不用等计时器）

![three duplicate ACKs indicates 3 segments received after a missing segment – lost segment is likely. So retransmit!](http://img.070077.xyz/202203160801774.png)

> Why 3?

### Flow Control

TCP provides flow control by having the *sender* maintain a variable called the **receive window**. TCP receiver “advertises” free buffer space in *rwnd* field in TCP **header** ，guarantees receive buffer will not overflow.

![RcvBuffer size set via socket options (typical default is 4096 bytes)](http://img.070077.xyz/202203160804150.png)

假设接收缓冲区已满(rwnd = 0)。若向接收方发送rwnd = 0报文丢失，而TCP只有在有数据要发送或者有应答要发送时才会向主机发送一个段。因此，发送方永远不会知道接收缓冲区中已经清空了一些缓冲区（被阻塞）。为解决这个问题，要求发送方在的接收窗口为零时继续发送窗口探测报文进行确认。

###  TCP Connection Management

#### 3-way handshake

> why not 2？无法防止历史连接的建立。
>
> | ![can’t “see” other side](http://img.070077.xyz/202203160809783.png) | ![delay->reordering?](http://img.070077.xyz/202203160808652.png) |
> | ------------------------------------------------------------ | ------------------------------------------------------------ |

1. The client-side TCP first sends a special TCP segment - **SYN segment** to the server-side TCP.
2. the server extracts the TCP SYN segment from the datagram, allocates the TCP buffers and variables to the connection, and sends a connection-granted segment - **SYNACK segment** to the client TCP.
3.  Upon receiving the SYNACK segment, the client also allocates buffers and variables to the connection. 

![](http://img.070077.xyz/202203160810032.png)

> 补充知识：半连接队列（SYN Queue)和全连接队列(accept Q)
>
> 服务器收到SYN请求会存储到SYN Q，收到第三次握手的ACK后，添加到AC Q
>
> - SYN洪泛攻击
>
> 通过SYN Cookie缓解。

#### 四次挥手

![TCP State: closed](http://img.070077.xyz/202203160813271.png)

> Why 4? Fin表示客户端不再发送数据，服务器返回ACK后可能依旧有待发送数据，处理后发送Fin
>
> 为什么最后需要Time wait 2MSL？防止旧连接的ACK包 、确保双工连接关闭。

## Principles of Congestion Control

若对网络中某一资源的请求超过了所能提供的可用部分，网络的性能（吞吐量、时延等）就要变差。这种情况就叫拥塞。

拥塞的代价：

![\lambda: payload/speed](http://img.070077.xyz/202203171814197.png)

- (queueing) delay increases as capacity approached
- loss/retransmission decreases effective throughput
- un-needed duplicates further decreases effective throughput
- upstream transmission capacity / buffering wasted for packets lost downstream

How to solve? Feedback.

![](http://img.070077.xyz/202203171817821.png)

## TCP Congestion Control

### Classic TCP Congestion Control

AIMD：Additive Increase and Multiplicative Decrease

- **A lost segment** implies congestion, and hence, the TCP sender’s rate should be
  decreased when a segment is lost.
- An acknowledged segment indicates that the network is delivering the sender’s
  segments to the receiver, and hence, the sender’s rate can be increased when an
  ACK arrives for a previously unacknowledged segment. 
- Bandwidth probing. 

​	![](http://img.070077.xyz/202203172251814.png)

> How AIMD?
>
> ![](http://img.070077.xyz/202203172255482.png)

1. Slow Start

   when connection begins, increase rate exponentially until first loss event.

   initially *cwnd* = 1 MSS, double *cwnd* every RTT done (every ACK received).

   when should this exponential growth end? loss, *cwnd* >= *ssthresh*, three duplicated ACK then fast retransmit.

   > *ssthresh*:“慢启动阈值”，默认为64KB
   >
   > ssthresh is half the value of cwnd when congestion was last detected.

2. Congestion Avoidance

   因为已经接近临界了，TCP 变得保守，increases the value of *cwnd* by just a single MSS every RTT, 直到发现拥塞。
   
3. Fast Recovery
  
  In fast recovery, the value of *cwnd* is increased by 1 MSS for every duplicate ACK received for the missing segment that caused TCP to enter the fast-recovery state.
  
  - 重传DACKs指定的数据包，如果再收到DACKs，那么cwnd大小增加一，如果收到新的ACK（重传成功），退出快速恢复算法。将cwnd设置为ssthresh，然后进入拥塞避免算法。
  
  ![example](http://img.070077.xyz/202203172316664.png)
  

### Better AIMD

![](http://img.070077.xyz/202203172320222.png)

![](http://img.070077.xyz/202203172320323.png)

### 网络辅助拥塞控制

![IP 报头（ToS 字段）中的两个位由网络路由器标记，以指示拥塞](http://img.070077.xyz/202203172342187.png)

- 基于延迟的拥塞控制：*Keep the pipe just full, but no fuller*

$RTT_{min}$  : minimum observed RTT (uncongested path)
uncongested throughput with congestion window cwnd is *cwnd*/$RTT_{min}$ 

###  Fairness

Yes.

- congestion avoidance: additive increase
- loss: decrease window by factor of 2

![](http://img.070077.xyz/202203172349607.png)

> When running over UDP, applications at a constant rate and occasionally lose packets, rather than reduce their rates to “fair” levels at times of congestion and not lose any packets.

> web browsers help us when multiple parallel connections, e.g.:
>
> link of rate R with 9 existing connections
>
> •new app asks for 1 TCP, gets rate R/10
>
> •new app asks for 11 TCPs, gets R/2 

## QUIC

Quick UDP Internet Connections Protocol`s major features:

- Connection-Oriented and Secure.

  ![](http://img.070077.xyz/202203180125838.png)

- Streams. QUIC allows several different application-level “streams” to be multiplexed through a single QUIC connection and establish fast（1 handshake）.

- Reliable, TCP-friendly congestion-controlled data transfer.

  ![](http://img.070077.xyz/202203180127668.png)
  
  a lost UDP segment only impacts those streams whose data was carried in that segment
  
# The Network Layer: Data Plane

## Overflow

- Forwarding：the router-local action of transferring a packet from an input link **interface** to the appropriate output link interface.
- Routing: the network-wide process that determines the end-to-end **paths** that packets take from source to destination

routers:

- examines header fields in all IP datagrams passing through it

- moves datagrams from input ports to output ports to transfer datagrams along end-end path

![control plane: Remote controller](http://img.070077.xyz/202203180831095.png)

### Network Service Model

The network service model defines the **characteristics** of end-to-end delivery of packets. 

**best-effort service** to：

- Guaranteed delivery (with guaranteed delivery). 
- In-order packet delivery. 
- Guaranteed minimal bandwidth. 
- Security. 

> No guarantees on：
>
> i.successful datagram delivery to destination
>
> ii.timing or order of delivery
>
> iii.bandwidth available to end-end flow
>
> 其实服务模型有很多种，其机制和复杂度的权衡，难以辨明优劣。尽力而为即可。

## What’s Inside a Router?

![](http://img.070077.xyz/202203180838150.png)

- Destination-based forwarding
- Generalized forwarding  

### IO Port Processing

![(Look up here)](http://img.070077.xyz/202203192108872.png)

the router uses the longest prefix matching rule in lookup table to decide the dest.

![](http://img.070077.xyz/202203192115611.png)

### Switching

![](http://img.070077.xyz/202203192110615.png)

-  Switching via a bus: must wait since only one packet can cross the bus at a time.
- Switching via an interconnection network: parallel(fragment datagram), A crossbar switch is **non-blocking**

More sophisticated interconnection networks use multiple stages of switching elements to allow packets from different input ports to proceed *towards* the same output port at the same time through the multi-stage switching fabric.

### Queuing

![](http://img.070077.xyz/202203192124595.png)

- Head-of-the-Line (HOL) blocking: queued datagram at front of queue prevents others in queue from moving forward

![buffer here](http://img.070077.xyz/202203192126370.png)

Datagrams can be lost due to congestion, lack of buffers（Buffering when arrival rate via switch *exceeds* output line speed ）

> The buffering size: （recommendation） $ \frac{RTT \times C}{\sqrt{n}} $
>
> too much buffering can increase delays

###  Packet Scheduling

- FIFO

- Priority Queuing

  ![](http://img.070077.xyz/202203192131091.png)

- Round Robin and Weighted Fair Queuing (WFQ)
  ![](http://img.070077.xyz/202203192132836.png)

## The Internet Protocol (IP)

### IPv4

![Datagram](http://img.070077.xyz/202203200142750.png)

The boundary between the host and the physical link is called an *interface*.

To determine the subnets, detach each interface from its host or router, creating islands of isolated networks. Each of these isolated networks is called a *subnet*.

The Internet’s address assignment strategy is known as Classless Interdomain Routing (CIDR)

![](http://img.070077.xyz/202203200452296.png)

### Obtaining Address

IP uses dotted-decimal notation and LPM.

![23：CIDR 掩码位数](http://img.070077.xyz/202203200458354.png)

- How does *host* get IP address?

  **DHCP**（ Dynamic Host Configuration Protocol） dynamically get address from as server， with plug-and-play or zeroconf.

![find DHCP by broadcast](http://img.070077.xyz/202203200504196.png)

![](http://img.070077.xyz/202203200504874.png)

DHCP server can also formulates a **encapsulated** DHCP ACK containing client’s IP address, IP address of first-hop router for client, name & IP address of DNS server。

- How a SOHO manage IP addresses? 

  **NAT**(Network Address Translation)

  - private addresses refers to a network whose addresses **only** have meaning devices within that network
  - just one IP address needed from provider ISP for *all* devices

![](http://img.070077.xyz/202203200550241.png)

### IPv6

![](http://img.070077.xyz/202203200553776.png)

> No fragmentation/reassembly: these operations can be performed only by the source
> and destination
>
> Header checksum: Transport Layer and link-layer have checked.

- How will the public Internet based on IPv4 be transitioned to IPv6? 

  **tunneling**。![physical view](http://img.070077.xyz/202203200557905.png)

## Generalized Forwarding

通过“匹配+动作”（match bits in arriving packet header(s) in any layers, take action），实现通用转发。下面是OpenFlow转发表的机制：

![Match](http://img.070077.xyz/202203201035397.png)

![Action](http://img.070077.xyz/202203201037720.png)

举例：

![](http://img.070077.xyz/202203201039149.png)

## MiddleBox

we’ve also encountered other network equipment (“boxes”) within the network that sit on the data path and perform functions other than forwarding.

![image-20220320110010301](http://img.070077.xyz/202203201100463.png)

- SDN: (logically) centralized control and configuration management often in  private/public cloud
- network functions virtualization (NFV): programmable services over white box networking, computation, storage



# BUPT Useful PPTs

![](http://img.070077.xyz/202203100803907.png)
