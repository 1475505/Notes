注：自顶向下方法对底两层介绍不深，本篇可能比较杂糅。

# 数据链路层

数据链路层负责将数据报从一个节点传输至物理相邻的节点。提供的服务有：

- framing, link access: adding header, trailer.（通过转义字符）Use MAC addresses in frame headers identify source, destination.
- reliable delivery between adjacent nodes.
- flow control.
- error detection and correction.
- half-duplex and full-duplex.

In every host, link layer implemented in network interface card (NIC, 网卡) or on a chip, which attaches into host's system buses

## Error

### 奇偶校验位

奇校验表示 $all\space data\space bits \oplus checkBit == 1$
偶校验类似。

### 汉明码

不多说了，直接看吧：[How Hamming Code Works](https://harryli0088.github.io/hamming-code/)

### CRC

Cyclic Redundancy Check (循环冗余检验），即传输二元组<D, R>.
- 发送方将数据比特串看成多项式的系数，补充r个（生成多项式长度，由传输协议觉得）0后，进行异或除法，商为校验信息 R。
- 接收方对生成多项式进行模2除法，余数不为0即认为有错，拒绝。
![G（X） = X4 + X3 + 1](http://img.070077.xyz/202206141242571.png)

---
两种链路（信道）：
- point-to-point：PPP for dial-up access.地址不是必需的
- broadcast (shared wire or medium)：used in 802.11 wireless LAN, 4G/4G. satellite.地址是必需的

## 点到点的数据链路协议

### HDLC

高级数据链路控制，面向*比特*，零比特填充实现同步透明传输。帧首尾标志：“01111110”。有三种类型的帧：
- 信息帧：包含上层数据；捎带确认、包含流量控制和差错控制信息
- 监督帧：ACK 或 NAK
- 无编号帧：数据链路层的管理和链路控制。也可以配置在一对多模型。

![](http://img.070077.xyz/202206142227153.png)

> SLIP ——串行线IP:
> 字节填充，首尾标志：0xC0，有替换机制。
> 没有差错控制，只支持IP，不能动态分配IP地址，没有身份认证

### PPP

Point-to-Point Protocol，面向字节，字节填充。支持多种网络层协议、用户身份认证、物理层可以是同步的或异步的；去除了差错纠正、流量控制（靠上层了）。

>  PPPoE (PPP over Ethernet, ADSL)：拨号上网...


PPP 的帧格式：
- 控制字段0x03表示无编号模式，对控制字符前面加上转移字符 0x7D
- 协议字段表示帧中的数据类型。如LCP（链路控制协议）、NCP（网络控制协议）、IP...
<img src="http://img.070077.xyz/202206142240836.png"/>

## multiple access protocols

有两种控制方法进行广播信道的协调，以避免发生冲突：一是使用信道复用技术（本节），一是使用 CSMA/CD 协议。

multiple access channel (MAC) has three board class:
- channel partitioning: divide channel into smaller “pieces” (time slots, frequency, code)
- random access: allow collisions, “recover” from collisions
- taking turns: nodes take turns, but nodes with more to send can take longer turns

![](http://img.070077.xyz/202206142316984.png)


### DMA

多路访问是信道划分思想的体现，有四种。

FDMA(frequency division multiple access)，each station assigned fixed frequency band.

![](http://img.070077.xyz/202206142322255.png)

==计算题==
将100Mbps分成10个10 Mbps，每个子信道的帧到达率为500帧/秒，服务率为1000帧/秒。平均时延 = 1/(1000-500)= 2 ms✔️

TDMA(时分多路访问)：即每一轮，每个站点得到固定长度（即帧发送时间）的时隙。

example: 6-station LAN, 1,3,4 have packets to send, slots 2,5,6 idle

![](http://img.070077.xyz/202206142321541.png)

统计时分复用是对时分复用的一种改进，不固定每个用户在时分复用帧中的位置，只要有数据就集中起来组成统计时分复用帧然后发送。

==计算题==
数据率为100 Mbps，平均帧长为 10,000比特，帧服务率µ =10000 帧/秒，帧到达率为 5000帧/秒。平均时延？ ~~1/(10000-5000)= 0.2 ms.~~✔️


> 复习:
> 波分复用：![](http://img.070077.xyz/202206150150543.png)
> 码分复用（CDM）：码分多址，每个基站可以在整个频谱上随时传输。

上述均为静态信道分配。可见，FDM 平均时延远大于单信道，TDM性能差，不适合突发性。

### Random access protocols

#### Slotted ALOHA

time divided into equal size slots (time to transmit 1 frame). Assume: all frames same size, nodes start to transmit only slot beginning.


![](http://img.070077.xyz/202206150233953.png)

-> when node obtains fresh frame, transmits in next slot.
- if no collision: node can send new frame in next slot
- if collision: node retransmits frame **in each subsequent slot with probability p** until success

max efficiency = 1/e = 37%.

> Pure ALOHA:
> no synchronization: when frame first arrives: transmit *immediately*
>  ![](http://img.070077.xyz/202206150238202.png)![](http://img.070077.xyz/202206150239262.png)最大吞吐量（信道利用率）: 18.4% 争用期为2段帧时。（只有经过争用期之后还没有检测到碰撞，才能肯定这次发送不会发生碰撞。）

#### CSMA

载波监听多路访问基本思想: 发送数据之前先监听信道 (载波监听), 站点尽量将易冲突期(VP)减小到一个发送时延.

simple CSMA（carrier sense multiple access）: listen before transmit.
- if channel sensed idle: transmit entire frame
- if channel sensed busy: defer transmission 

在冲突时，1-坚持 CSMA 低负载下高吞吐量低时延；
非坚持 CSMA：高负载下高吞吐量

CSMA/CD: with collision detection:
- collisions detected within short time
- colliding transmissions aborted, reducing channel wastage
- collision detection easy in wired, difficult with wireless
(如果检测到冲突，立即停止传输 (退避) 并开始发送一个强化信号(jam信号)，而不是将整个帧都传输完。Jam信号传输结束后，随机等待一段时间，再重新开始监听信道。)

这个等待的时间采用 **截断二进制指数退避算法** 来确定。从离散的整数集合 $$ 0, 1, .., 2^{k}-1, k = min(尝试的次数, 10) $$中随机取出一个数，记作 r，然后取 r 倍的争用期作为重传等待时间。再次传输重试，最大延迟时间加倍，重传时间 = r * 基本重传时间。如果达到预设的最大尝试次数(通常为16次)，则终止。

协议使得帧发送时延远大于传播时延时，信道效率高。

### “Taking Turns"

属于无冲突协议。
- polling： master node “invites” other nodes to transmit in turn
- token passing: control token passed from one node to next sequentially.

#### 无线局域网协议

> 先介绍一下无线局域网
> WLAN: IEEE 802.11 (b: 2.4-5 GHz, a: 5-6 GHz)
> 无线宽带网： IEEE 802.16 (WiMax)
> 蓝牙: IEEE 802.15
> 
> 多路访问技术均采用CSMA/CA.(拥塞避免)
> 组网模式均支持基站方式和自组织方式.

| 隐蔽站问题(A->B && C->B) | 曝露站问题(B->A && A cant see D)) |
| ----------- | ------------|
| ![](http://img.070077.xyz/202206150300283.png)|![](http://img.070077.xyz/202206150300671.png)

- PCF (点协调功能)：轮询、集中控制；基站周期性广播beacon frame(信标帧)，控制小区内的所有活动。
- DCF (分布协调功能)： 站点使用握手机制来确定谁可以发送数据。
- (术语)RTS：请求发送帧，为了提醒源站信号范围内的站； CTS：发送帧，为了提醒目的站信号范围内的站。

MACA 实现：带冲突避免的多路访问。即用RTS/CTS预约信道。与CSMA/CD的退避算法类似。

![](http://img.070077.xyz/202206150305610.png)

## LANS

MAC (Media Access Control Address or LAN or physical or Ethernet) address: 
     - function: used “locally” to get frame from one interface to another physically-connected interface (same subnet, in IP-addressing sense).  
	     - 48-bit MAC address (for most LANs) burned in NIC ROM, also sometimes software settable. each interface on LAN has unique MAC address。（由 IEEE 统管分发）

一台主机拥有多少个网络适配器就有多少个 MAC 地址。例如笔记本电脑普遍存在无线网络适配器和有线网络适配器，因此就有两个 MAC 地址。

### 地址解析协议 ARP

网络层实现主机之间的通信，而链路层实现具体每段链路之间的通信。因此在网络通信过程中，IP 数据报的源地址和目的地址始终不变，而 MAC 地址随着链路的改变而改变。**ARP 实现由 IP 地址得到 MAC 地址。** 每个主机都有一个 ARP 高速缓存，里面有本局域网上的各主机和路由器的 IP 地址到 MAC 地址的映射表`< IP address; MAC address; TTL>`。主机可通过广播的方式发送 ARP 请求分组，主机 B 收到该请求后会发送 ARP 响应分组给主机 A 告知其 MAC 地址。

> TTL (Time To Live): time after which address mapping will be forgotten (typically 20 min)

#### Routing to another subnet

- 编址

R determines outgoing interface, passes datagram with IP source A, destination B to link layer and creates link-layer frame containing A-to-B IP datagram. Frame destination address: B's MAC address

![](http://img.070077.xyz/202206150854450.png)

### Ethernet

- 物理模型：物理上的星型拓扑, 逻辑上的总线型拓扑。早期使用集线器进行连接，集线器是一种*物理层*、半双工通信设备， 作用于比特（而不是帧），当一个比特到达接口时，集线器重新生成这个比特，并将其能量强度放大，从而扩大网络的传输距离，之后再将这个比特发送到其它所有接口。如果集线器同时收到两个不同接口的帧，那么就发生了碰撞。all nodes in same collision domain (can collide with each other). 目前以太网使用交换机替代了集线器。

> Repeaters(中继器)：更早的物理层设备，放大信号、对信号整形，以减小信号失真和衰减，使
信号传播得更远，扩展以太网段。

- 帧格式：![](http://img.070077.xyz/202206150858200.png) 
前导码: 8字节, 101010…101011, 用于时钟同步；数据长度在 46-1500 之间，如果太小则需要填充。因此，最小帧长: 64字节。为了使主机有时间检测到冲突，帧的发送时间应该 ≥ 2 * 电缆的传播时延（Propagation delay,  $\tau$ ，即CSMA节中提到的争用期）

- 传输：
  - connectionless: no handshaking between sending and receiving NICs 
  - unreliable: receiving NIC doesn’t send ACKs or NAKs to sending NIC
  - 帧间隙（IFG）：用于站点从发送模式转换为接收模式。10M bps 以太网的帧间隙是9.6μs。
  - 使用1-坚持的CSMA/CD。

==计算题：IEEE 802.3的性能==
![](http://img.070077.xyz/202206150919015.png)

- 交换机(Switch) is a link-layer device, examine incoming frame's **MAC** address, *selectively* forward  frame to one-or-more outgoing links when frame is to be forwarded on segment, uses CSMA/CD to access segment. each link is its own collision domain.(全双工通信) 交换机具有自学习能力，学习的是交换表的内容，交换表中存储着 MAC 地址到接口的映射。即插即用，不需配置。
> 自学习过程:
> 主机 A 向主机 B 发送数据帧时，交换机把主机 A 的接口的映射写入交换表中。如果交换表没有主机 B 的表项，那么主机 A 就发送广播帧Flooding，主机 B 向 A 回应该帧。
> MAC转发表也是本地的，只在单个广播域（即一个IP子网内）有效。

- 千兆以太网
最大线缆长度仅有25米，对CSMA/CD协议进行载波扩展，达到200米：在普通的帧后面增加填充比特，将帧的长度扩展到512 字节； 允许发送方将多个帧级联在一起一次传输出去。

- 流量控制：快速以太网和千兆以太网很容易导致缓冲区溢出。使用暂停帧 (一种控制帧PALSE) 允许或禁止帧的传输，暂停时间是最小帧时的整数倍。

![](http://img.070077.xyz/202206150930866.png)

### 网桥

工作在数据链路层、即插即用、用软件实现帧处理的网络互连设备，基于帧的目的MAC地址来决定将帧转发到相邻的LAN（一次只能分析和转发一个帧，而交换机可以并行），支持协议翻译等。

学习网桥工作在混杂模式，可以获得其所连接的各LAN上的每个帧，通过站表进行帧的路由和转发。如果查到的输出端口与输入端口一致，则不转发；如果输出端口与输入端口不一致，则转发到相应端口；如果查不到，则洪泛转发到除输入端口之外的所有端口。
![](![](http://img.070077.xyz/202206151049174.png)
<img src="http://img.070077.xyz/202206151049174.png"/>

生成树网桥用一棵可以到达每个网桥的生成树覆盖实际的拓扑结构。根据802.1d协议构造生成树：每个网桥周期地从它的所有端口广播一个配置消息给邻居，选择具有最低标识符的网桥作为生成树的根，消息交换到最终所有网桥将都同意一个根。记住找到的到根的最短路径，关闭不属于最短路径的一部分的端口，自动检测拓扑变化并更新生成树。

### VLAN

虚拟局域网可以建立与物理位置无关的逻辑组，只有在同一个虚拟局域网中的成员才会收到链路层广播信息。

上述协议中，广播帧会非常频繁地出现，即产生广播风暴。因此可分割广播域（一般都必须使用到路由器），可以以路由器上的网络接口(LAN Interface)为单位分割广播域。
![](http://img.070077.xyz/202206151042949.png)

- port-based VLAN: switch ports grouped (by switch management software) so that single physical switch. support traffic isolation

## 链路虚拟化

Multiprotocol label switching (MPLS，多协议标签交换)，客观上讲是一种分组交换的虚电路网络。基于标签执行交换，而不必考虑分组的 IP 地址 。
high-speed IP forwarding among network of MPLS-capable routers, using **fixed** length label (instead of shortest prefix matching). 
![](http://img.070077.xyz/202206151103995.png)
MPLS 提供了沿着多条路由转发分组的能力，进行流量引导。这是流量工程的研究范畴。

> 在数据中心中，数据中心通常应用路由器和交换机等级结构，有一些有趣的机制保障可用性。
> - 负载均衡器：![](http://img.070077.xyz/202206151111504.png)
> - 为了降低数据中心的费用，同时提高其在时延和吞吐量上的性能，部署能够克服传统等级设计缺陷的新型互联体系结构和网络协议。如：全连接拓扑，高度互联，替代交换机和路由器的等级结构；模块化数据中心，我感觉是像集群服务。

# 物理层
（速通北邮PPT）

物理层是协议模型中的最底层，定义了bit信号在信道上传输时相关的电气、时序和其他接口。

术语：
- 信道：传送信息的媒体（介质）
- 带宽：指接收能量能够保留至少一半的频率范围(单位：Hz)
- **波特率(Baud): 每秒中传输的码元（即信号单元）个数** =1/T（T是一个信号单元的周期）
- 比特率(bps) = 波特率 * $log_2V$ ，V是信号的有效状态数(电平级数)。表示每秒中传输的比特（数据位）数。
- 吞吐量：网络容量的度量，表示单位时间内网络可以传送的数据位数（bps）

- 奈奎斯特公式：**无噪声信道**最大数据速率 = 2 * B * $log_2V$ （B - 带宽）
- 香农公式：**有噪信道**最大数据率 = $B \times log_2(1 + \frac{S}{N})$
- 信噪比SNR：$S/N_{db} = 10log_{10}\frac{S}{N}$ ，单位：分贝（dB）
![](http://img.070077.xyz/202206151238884.png)

![](http://img.070077.xyz/202206151253501.png)

## 无线传输分类

- 无线电的传播
![](http://img.070077.xyz/202206151253852.png)

- 微波传输(100M~10GHz)：多径衰落是微波通信的一个很严重的问题。
- 红外传输
- 通信卫星(大型微波中继器)：包含多个天线和多个转发器，转发器监听频谱中的某一部分。GEO具有较大的往返延迟时间。 ![](http://img.070077.xyz/202206151245168.png)
## 数字调制和编码

- QPSK：正交相移键控
- QAM：正交幅度调制

星座图：相位是夹角，振幅是到原点的距离。

- 不归零 (NRZ-L)
- 逆转不归零(NRZI)
- **Manchester 曼彻斯特编码**：0：在位时间中从高到低跳转；1：低到高。IEEE 802.3局域网使用
![](http://img.070077.xyz/202206151248097.png)

## PSTN电话系统

DSL: 数字用户线。

- ADSL: 离散多音调制：将1.1 MHz划分为256个4312.5 Hz的信道，可用250条信道(FDM)：上行流控制 (1) + 下行流控制 (1) + 用户数据 (248)。 采样速率是4000波特。
- DMT:离散多音调制

三个设备：
-  数字发送器---线路编码技术，数字基带信号，抗干扰、带宽效率
-  调制解调器---调制解调技术，信号频率变换，通带信号，长距离传输、有效利用带宽
-  编码解码器---采样、量化、编码技术，将模拟信号数字化，将模拟信源的输出变为数字数据



---
参考：
北邮《计算机网络》PPT
[CS-Notes (cyc2018.xyz)](http://www.cyc2018.xyz/)
[《计算机网络 - 自顶向下方法》第八版](https://gaia.cs.umass.edu/kurose_ross/index.php)
[Macsen's Blog](https://www.macsen.xyz/2021/03/19/%e7%ac%ac%e4%ba%8c%e7%ab%a0-%e6%95%b0%e6%8d%ae%e9%80%9a%e4%bf%a1%e5%9f%ba%e7%a1%80/)