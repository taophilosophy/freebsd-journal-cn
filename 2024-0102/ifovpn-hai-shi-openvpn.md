# if\_ovpn 还是 OpenVPN

由克里斯托夫·普罗沃斯特 (Kristof Provost) 创作

今天~1~，你将要阅读~2~有关 OpenVPN 的 DCO。

最初由 James Yonan 开发，OpenVPN 于 2001 年 5 月 13 日首次发布。它支持许多常见平台（如 FreeBSD，OpenBSD，Dragonfly，AIX，...）和一些不太常见的平台（macOS，Linux，Windows）。它支持点对点和客户端-服务器模型，具有基于预共享密钥、证书或用户名/密码的身份验证。

正如您对任何存在了 20 多年的项目所期望的那样，它为许多不同的用例增加了许多功能。

## 定制

虽然 OpenVPN 非常好，但显然必须有问题。没有问题，这篇文章就不会那么有趣了~3~。事实上确实存在问题，即 OpenVPN 是作为单线程用户空间进程实现的。

它使用 if_tun 将数据包注入网络堆栈。因此，其性能没有跟上当前的连接速率。它还使得难以利用现代多核硬件或加密卸载硬件的优势。

![](https://freebsdfoundation.org/wp-content/uploads/2024/02/provost_fig1.png)

OpenVPN 的主要问题在于其用户空间性质。传入的流量通常由网卡接收，然后将数据包 DMA 到内核内存中。然后，进一步由网络堆栈处理，直到确定数据包属于哪个套接字，并将其传递到用户空间。这个套接字可能是 UDP 或 TCP。

将数据包传递到用户空间涉及复制它，这时用户空间的 OpenVPN 进程验证和解密数据包，并使用 if_tun 将其重新注入到网络堆栈中。这意味着将明文数据包重新复制回内核进行进一步处理。

不可避免地，所有这些上下文切换和来回复制对性能产生了显着影响。

在当前的架构中，要显著提升性能非常困难。

## 什么是 DCO

现在我们已经确定了我们的问题，我们可以开始思考解决方案 ~4~。

如果我们的问题是用户空间的上下文切换，那么一个合理的解决方案是将工作留在内核中，这就是 DCO-数据通道卸载-的作用。

![](https://freebsdfoundation.org/wp-content/uploads/2024/02/provost_fig2.png)

DCO 将数据通道，即加密操作和流量隧道，移入内核。它通过一个新的虚拟设备驱动程序 if_ovpn 来实现这一点。OpenVPN 用户空间进程仍然负责连接设置（包括身份验证和选项协商），通过一个新的 ioctl 接口与 if_ovpn 驱动程序协调。

OpenVPN 项目决定引入 DCO 是一个去除一些遗留功能并进行一些一般整理的好机会。作为这个过程的一部分，他们采取了亨利·福特对加密算法选择的方法。您可以选择任何您喜欢的算法，只要您喜欢 AES-GCM 或 ChaCha20/Poly1305。除了黑色之外。DCO 还不支持压缩、第 2 层流量、非子网拓扑或流量整形～5~。

在这里需要注意的是，DCO 不会改变 OpenVPN 协议。客户端可以在不支持它的服务器上使用，反之亦然。当然，双方都使用它时效果最好，但这并非必须要求。

## 考虑因素

这是我大谈特谈这一切有多么困难的部分，所以你们会对我真的做到了感到印象深刻。如果我告诉你我正在做这件事，这仍然有效吗？让我们找出来！

无论如何，有几件事需要特别注意：

## 多路复用

第一个问题是，OpenVPN 使用单个连接传输隧道数据和控制数据。隧道数据需要由内核处理，而控制数据由 OpenVPN 用户空间进程处理。

您可以看到该问题。套接字最初由 OpenVPN 本身打开并完全拥有。它设置隧道并处理身份验证。完成后，它部分地将控制权移交给内核端（即 if_ovpn）。

这意味着通知 if_ovpn 文件描述符（内核用于查找内核结构套接字的文件描述符），以便它可以持有对其的引用。这确保了内核在使用套接字时不会消失。可能是因为 OpenVPN 进程已终止，或者因为它今天心情不好而决定与我们作对。这是用户空间，它会做些疯狂的事情。

对于那些想要跟踪内核代码的人来说，你们要找的是 ovpn_new_peer() ~6~ 函数。

在查找套接字后，我们现在也可以通过 udp_set_kernel_tunneling() 安装过滤函数。 过滤器 ovpn_udp_input() 查看指定套接字的所有传入数据包，并确定它是否是应该处理的有效载荷数据包，还是用户空间中的 OpenVPN 应该处理的控制数据包。

这个隧道功能也是我必须对网络堆栈做出的唯一更改。 它需要被告知某些数据包将由内核处理，其他数据包仍然可以传递到用户空间。 在 https://cgit.freebsd.org/src/commit/?id=742e7210d00b359d81b9c778ab520003704e9b6c 中完成了此操作。

ovpn_udp_input() 函数是接收路径的主要入口点。 网络堆栈将到达已安装的套接字的任何 UDP 数据包移交给这个函数。

内核驱动程序首先检查数据包是否可以处理。也就是说，数据包是数据包并且它是为已知对等方 ID 的目的地。如果不是这种情况，则过滤函数告诉 UDP 代码将数据包传递到正常流程，就好像没有过滤函数一样。这意味着数据包将到达套接字，并由 OpenVPN 的用户空间进程处理。

DCO 驱动程序的早期版本具有用于读取和写入控制消息的单独 ioctl 命令，但 Linux 和 FreeBSD 驱动程序都已经调整为使用套接字。这简化了对控制包和新客户端的处理。

另一方面，如果数据包是已知对等方的数据包，则对其进行解密，验证其签名，然后将其传递到网络堆栈进行进一步处理。

对于跟进的人，可以在此处找到 https://cgit.freebsd.org/src/tree/sys/net/if_ovpn.c?id=da69782bf06645f38852a8b23af#n1483 。

## UDP

OpenVPN 可以在 UDP 和 TCP 上运行。虽然 UDP 是三层 VPN 协议的明显选择，但有些用户需要通过 TCP 运行它以穿越防火墙。

FreeBSD 内核为 UDP 套接字提供了一个方便的过滤函数，但对于 TCP 却没有等效功能，因此 FreeBSD if_ovpn 目前仅支持 UDP 而不支持 TCP。

Linux DCO 驱动程序开发人员则更为勇敢，选择实现了 TCP 支持。开发人员事实上克服了种种困难，并因此获得了显著的成长。

## 硬件加密卸载

if_ovpn 依赖于内核 OpenCrypto 框架进行加密操作。这意味着它还可以利用系统中存在的任何加密卸载硬件。这可以进一步提高性能。

它已经与 Intel 的 QuickAssist 技术（QAT）、SafeXcel EIP-97 加密加速器和 AES-NI 进行了测试。

## 锁定设计

看，如果你认为你可以讨论内核代码而不谈论锁定，那我不知道该告诉你什么。这对你来说是天真乐观的。

几乎每个现代 CPU 都有多个核心，能够使用不止一个核心会很好。也就是说，在一个核心在工作时，我们不能把其他核心锁定在外面。这是不礼貌的。而且性能也不好。

幸运的是，这事情做起来相当容易。整个方法基于区分对 if_ovpn 内部数据结构的读写访问。也就是说，我们允许许多不同的核心同时查找事物，但只允许一个核心更改事物（并且在进行更改时不允许任何读者）。这种方法效果还不错，因为大多数情况下我们不需要更改事物。

普通情况下，当我们接收或发送数据包时，只需要查找键、目的地地址和ports以及其他相关信息。

仅当我们修改事物（即配置更改或重新选择密钥）时，我们需要进行写锁定，并且我们会暂停数据通道。这种暂停足够短暂，我们这些微不足道的人类大脑不会注意到，这让每个人都很开心。

这个“我们不对处理数据进行更改”的规则有一个例外，那就是数据包计数器。每个数据包都会被计数（甚至两次，一次是数据包计数，一次是字节计数），这必须同时进行。同样地，我们很幸运，因为内核的 counter(9) 框架恰好就是为这种情况而设计的。它会保持每个 CPU 核心的总数，因此一个核心不会影响或减慢另一个。只有当实际读取计数器时，它才会要求每个核心报告其总数并将它们相加。

## 控制界面

每个 OpenVPN DCO 平台都有其自己独特的方式在用户空间的 OpenVPN 和内核模块之间进行通信。

在 Linux 上，这是通过 netlink 完成的，但 if_ovpn 的工作是在 FreeBSD 的 netlink 实现准备好之前完成的。由于我仍在因上次违规行为而试用期，我决定使用其他方式代替。

通过现有的接口 ioctl 路径配置 if_ovpn 驱动程序。具体来说， SIOCSDRVSPEC/SIOCGDRVSPEC 调用。

这些调用通过一个 struct ifdrv 传递给内核。ifd_cmd 字段用于传递命令，ifd_data 和 ifd_len 字段用于在内核和用户空间之间传递设备特定的结构。

if_ovpn 有些偏离传统的方法，它传输序列化的 nvlists 而不是结构体。这使得扩展接口变得更加容易。或者更确切地说，这意味着我们可以在不破坏现有用户空间消费者的情况下扩展接口。如果向结构体添加新字段，其布局会发生变化，这意味着现有代码可能会因为大小不匹配而拒绝接受它，或者会因为字段不再具有之前的含义而变得非常困惑。

序列化的 nvlists 允许我们添加字段，而不会让对方困惑。任何未知的字段将会被忽略。这使得添加新功能变得更加容易。

## 路由查找

你可能认为 if_ovpn 不需要担心路由决策。毕竟，内核的网络堆栈在数据包到达网络驱动程序时已经做出了路由决策。你错了。我本可以嘲笑你的，但我也花了一些时间才弄明白这一点。

问题在于在给定 if_ovpn 接口上可能有多个对等体（例如，当它作为服务器并有多个客户端时）。内核已经确定了所讨论的数据包需要发送到其中之一，但内核基于这样的假设运行，即所有这些客户端都存在于单个广播域中。也就是说，发送到接口上的数据包对所有客户端都是可见的。然而，在这种情况下并非如此，因此 if_ovpn 需要确定数据包必须发送到哪个客户端。

这由 ovpn_route_peer() 处理。此函数首先查看对等体列表，以查看是否有任何对等体的 VPN 地址与目标地址匹配（由 ovpn_find_peer_by_ip() 或 ovpn_find_peer_by_ip6() 完成，取决于地址族）。如果找到匹配的对等体，则将数据包发送到此对等体。如果没有找到，则 ovpn_route_peer() 执行路由查找，并使用结果的网关地址重复对等体查找。

只有当 if_ovpn 确定了要发送数据包的对等体后，才能对其进行加密并传输。

## 密钥轮换

OpenVPN 会不时更改用于保护隧道的密钥。这是 if_ovpn 留给用户空间的艰巨任务之一，因此需要在 OpenVPN 和 if_ovpn 之间进行一些协调。

OpenVPN 将使用 OVPN_NEW_KEY 命令安装新密钥。每个密钥都有一个 ID，并且每个数据包都包含用于加密的密钥 ID。这意味着在进行密钥轮换期间，所有数据包仍然可以解密，因为旧密钥和新密钥都是已知的并且在内核中保持活动状态。

安装新密钥后，可以使用 OVPN_SWAP_KEYS 命令使其生效。换言之，新密钥将用于加密出站数据包。

一段时间后，可以使用 OVPN_DEL_KEY 命令删除旧密钥。

## vnet

是的，我们将不得不谈论 vnet。我正在写这个，这是不可避免的。

我懒得完全解释，所以我只会指向一篇由更好的作者 Olivier Cochard-Labbé撰写的文章：“Jail：通过示例学习 vnet” ~8~。

将jails想象成将其变成具有自己 IP 堆栈的虚拟机。

这对于 pfSense 用例并非绝对必需，但它使测试变得更加容易。这意味着我们可以在单台机器上进行测试，而无需任何外部工具（除了 OpenVPN 本身，出于显而易见的原因）。

对于那些对此如何完成感兴趣的人，还有另一篇可能会有用的 FreeBSD Journal 文章：“自动化测试框架”，作者是...等等，我想我认识那个家伙，Kristof Provost。

## 性能

经历了这一切之后，我敢打赌你一定在问自己：“这真的有帮助吗？”

幸运的是，对我来说：是的，确实有帮助。

我在 Netgate 的一位同事花了一些时间，轻轻地用 iperf3 测试了一个 Netgate 4100 设备，得到了以下结果：

“if_tun” 是旧版 OpenVPN 的方法，没有使用 DCO。值得注意的是，它在用户空间中使用了 AES-NI 指令，而“DCO 软件”设置则没有。尽管如此，DCO 仍然稍微更快。在公平竞争的条件下（即 DCO 使用 AES-NI 指令的情况下），结果毫无悬念，DCO 的速度是原来的三倍多。

英特尔也有一些好消息：他们的 QuickAssist 卸载引擎甚至比 AES-NI 更快，使得 OpenVPN 的速度比以前快五倍。

## 未来工作

没有什么是那么好，不能再改进的，但在某些方面，这个下一个增强是 DCO 设计成功的结果。在线 OpenVPN 协议使用 32 位初始化向量（IV），因为加密原因我在这里不会解释，重复使用具有相同密钥的 IV 是个坏主意。

这意味着在达到那一点之前，必须重新协商密钥。OpenVPN 的默认重新协商间隔是 3600 秒，加上 30%的安全边际，这将转化为 2^32 * 0.7 / 3600，约为每秒 835，000 个数据包。那只是“仅仅”为 8 到 9 Gbit/s（假设每个数据包为 1300 字节）。

有了 DCO，这已经基本在当代硬件的范围之内。

尽管这是一个好问题，但仍然是一个问题，因此 OpenVPN 开发人员正在研究一种更新的数据包格式，将使用 64 位 IV。

## 谢谢

if_ovpn 的工作由 Rubicon Communications（以 Netgate 为品牌）赞助，用于其 pfSense 产品系列。自 22.05 pfSense Plus 发布约 12 后，它已在那里使用。这项工作已上游到 FreeBSD，并已成为最新的 14.0 版本的一部分。需要 OpenVPN 2.6.0 或更新版本才能使用。

我也想感谢 OpenVPN 开发人员，在初次出现 FreeBSD 补丁时非常热情，如果没有他们的帮助，这个项目不会取得现在这么好的进展。

## 脚注：

1. 或。无论你什么时候读到这里。
2. 好的。写作。阅读。看吧，如果你对这个过于追究准则，我们会在这里呆上整整一天。
3. 看吧，如果你对 DCO 不感兴趣，你可以去看下一篇文章。我相信它会很好的。
4. 我说“我们”，但尽管我很想为这个解决方案负责，但是 OpenVPN 开发人员想出了 DCO 架构并在 Windows 和 Linux 上实现了它。我所做的就是跟着他们的脚步，但是用于 FreeBSD。
5. 在 OpenVPN 中，DCO 可以与操作系统的流量整形（即 dummynet）结合使用。
6. [https://cgit.freebsd.org/src/tree/sys/net/if_ovpn.c?id=da69782bf06645f38852a8b23af#n490](https://cgit.freebsd.org/src/tree/sys/net/if_ovpn.c?id=da69782bf06645f38852a8b23af#n490)
7. 你可能也会说是因为这个结构变胖了。你可能会，我对此太客气了。
8. [https://freebsdfoundation.org/wp-content/uploads/2020/03/Jail-vnet-by-Examples.pdf](https://freebsdfoundation.org/wp-content/uploads/2020/03/Jail-vnet-by-Examples.pdf)
9. [https://freebsdfoundation.org/wp-content/uploads/2019/05/The-Automated-Testing-Framework.pdf](https://freebsdfoundation.org/wp-content/uploads/2019/05/The-Automated-Testing-Framework.pdf)
10. [https://shop.netgate.com/products/4100-base-pfsense](https://shop.netgate.com/products/4100-base-pfsense)
11. 主要是因为我自己也不理解它们。
12. [https://www.netgate.com/blog/pfsense-plus-software-version-22.05-now-available](https://www.netgate.com/blog/pfsense-plus-software-version-22.05-now-available)

KRISTOF PROVOST 是一名自由职业的嵌入式软件工程师，专注于网络和视频应用。他是 FreeBSD 的提交者，负责 FreeBSD 中 pf 防火墙的维护。目前，他大部分时间都在为 Netgate 的 pfSense 工作。

Kristof 有一个不幸的倾向，就是经常碰到 uClibc 的 bug，并且对 FTP 有强烈的厌恶。不要跟他谈论 IPv6 分片问题。