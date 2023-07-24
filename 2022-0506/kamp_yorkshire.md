# 我不是来自约克郡的，我保证！

- 原文: I’m Not from Yorkshire,I Promise!
- 作者：POUL HENNING-KAMP
- 译者：basebyte 【】为译者注

我这篇文章的初稿保存在文件名为“Unix_from_Yorkshire.txt”的文件中，随着文章变得越来越长，越发像是 UNIX 版本的蒙提·派森的“四个约克郡人”小品：

> 在一颗 Zilog Z8001 16 位 CPU 和 4MB 内存的计算机上运行整个石油大亨游戏？
> 使用 103 张单面单密度软盘安装 UNIX 操作系统，但是第 92 张软盘是坏的？
> 使用 1.2kbps 的调制解调器？
> 没有源代码？
> TZ 规则的二进制补丁？
> 电子邮件地址中带有感叹号？
> 使用 X.25 网络而不是互联网？

但当你要试着告诉现在的年轻人…… 他们是不会相信的。

他们可能也不会读它，因为虽然这样的故事在痛饮啤酒时可以提供不错的娱乐，但在白天阅读时会显得枯燥乏味而单调。

但我想我应该介绍一下自己：

我是 phk@FreeBSD.org，我的笔记本电脑叫做“critter”，已经运行了 FreeBSD-current 接近 30 年。在最近十多年里，平均每 18 小时我都会提交代码到 FreeBSD 源代码库，只有在 Norvegian Newspaper 遇到 HTTP 性能问题时我才会停止提交代码。

**如何让 UNIX 不那么糟糕？**

我最终选择 FreeBSD 是因为 UNIX 真的很糟糕，而我擅长让它变得不那么糟糕，并且决心这样做。

我熟悉的 UNIX 版本在 1990 年左右已经被某些小型或微型计算机供应商改得面目全非，他们跟风加入 UNIX 阵营，承诺提供“可移植的 UNIX”，然后竭力用他们自己的“增强功能”将用户束缚住。

一般来说，供应商的支持毫无用处。在许多情况下，他们对 UNIX 的了解不如他们的客户。

在我接触过的十几台 UNIX 机器中，没有一台不让我不得不拆解某些程序或内核，以便找出问题所在，往往还不得不进行二进制补丁，才能使事情正常运行。

在 1992 年春天，386BSD 0.0 版本发布时，我下载并尝试过，但它太不稳定了，对我来说没有多大用处。不过，在工作时我打印了很多它的源代码，以便在晚上回家的火车上能够阅读。

到了 1992 年夏天，386BSD 0.1 版本发布了，紧接着没过几天，又发布了 386BSD“0.1-newer”，看起来更有前途了。

为了对其进行测试，我将其安装在一个被丢弃的“UNISYS 专业型 UNIX 工作站”上（这是一台价格高得离谱的 486 PC，配有 QIC-120 盒式磁带机和 UNIX 二进制许可证，按现在的价格计算，其市价超过 1 万美元），然后将它运到 PBX 机房，让它用于记录和统计流量。

作为未来的预兆，版本发布两周后，我在 comp.unix.bsd 的第一篇文章就是关于磁盘驱动器几何结构问题，因为那时存在臭名昭著的 1024 柱面限制。

在 9 月份，我提交了第一个补丁，也是 Terry Lambert 的“非官方 386BSD 补丁套件”中的第 19 个和最后一个补丁：

> PATCH00019
> 补丁：清理 SLIP 接口以避免挂起
> 作者：Poul-Henning Kamp (p...@data.fls.dk)
> 描述：
> 这是一个用于清理终端驱动程序（特别是 com 驱动程序）与 sl#接口之间关系的补丁，这不是一个权宜之计，而是一个真正的错误修复。
> 症状：在多次出现“com#: silo overflow”后，SLIP 将停止工作。
> 问题概述：SLIP 接口会忽略终端驱动程序的任何问题通知（奇偶校验错误、帧错误或溢出），这基本上意味着可以立即丢弃数据包。另外，在数据包组装中的溢出情况将相对较少被注意到。

正如标题所说，我不是来自约克郡的。我来自丹麦最大岛屿锡耶兰的西南角，那里既落后又贫穷。

但在 1992 年，我家里的电脑上已经有了 TCP/IP 连接。

我向公司的回拨安全终端拨号，等待它回拨，按下 V.22bis 调制解调器上的按钮，登录到公司的 386BSD 机器，在那里启动 SLIP，再在我的 PC 上启动 SLIP，公司的电脑已经与丹麦 Unix 用户组网络 "DKNet "建立了连接，这样通过连接公司的电脑，我就可以访问互联网了。

最开始，每秒 200 字节的速度下生活还算不错，但接着事情变得奇怪起来。

Bill Jolitz 移植了 NET/2 的源代码，并添加了使其能在 i386 上运行的必要代码，但他并没有对许多向他提供帮助和补丁的人做出回应，承诺的 0.2 版本从未实现，而补丁包却在不断增加，最终达到 138 个 "官方 非官方 "补丁，其中 3 个半是我贡献的。

在 1993 年春季，Chris G. Demetriou 等人放弃了对 Bill Joliz 的等待，创建了“NetBSD”项目，开始一个新的 CVS 树，其中包含 386BSD+补丁套件。

然后，三个月后，也就是 30 年前，Jordan Hubbard 等人做了完全相同的事情，取名为 FreeBSD。

**为何会这样？**

我也无从知晓：在那个时候，我身处地球的另一边，并且只是一个非常小的贡献者。

对我而言，NetBSD 似乎更具理想主义色彩：

他们希望得到一个高度可移植的、纯正的 BSD UNIX，可以在各种硬件上运行：
比如 HP、Sun、Digital、IBM PC 等，而这其中我既没有接触的机会，也对此不感兴趣。

NetBSD 团队也似乎更加学术化，更关注做事情的“正确性”，而不是追求快速实现功能。

平心而论： 他们非常成功地完成了自己的目标，所以在此向前辈们致以崇高的敬意。

FreeBSD 专注于 i386 PC 平台，这也是我能接触到的平台，因此我倾向于 FreeBSD 项目，因为那里的人似乎也像我一样 "以生产为导向"。

1993 年 11 月底，FreeBSD 1.0 发布，我在其中打了足够多的补丁，因此被列入 "FreeBSD 附加贡献者 "名单，几周后我甚至收到了 Walnut Creek 公司免费赠送的光盘。

1994 年 2 月 26 日，几乎就在我写这篇文章的 29 年前，乔丹授予了我 FreeBSD 项目的第 21 个提交者的荣誉称号。

**你的使命，如果你选择接受它**

根据 CVS 日志，乔丹这样写道： "......这样他【指本文作者：POUL HENNING-KAMP】就可以帮我清理 ports【的 bug】了"，而 1.1.5.1-tree 的 CVS 提交日志就是我的见证： 在我不慌不忙地大步走进 src，提交了一堆我对内核和 NTPD 的费时费力的补丁之前，我确实对 ports 做了两次提交。

接着，我提交了一组一团糟的 Shell 和 Tcl 混合脚本，可以将 GCC 2.5.8 编译器源代码树转换为 FreeBSD Makefiles 所期望的形式，以便其他人想要从 GCC 1.39 升级时使用。

同时，我也在洽谈一份新的工作，作为一个现场软件工程师，为一个非常大的文档影像系统提供支持，该系统将用于“Giro”支票的后处理，由“TRW Financial Services”从加利福尼亚州奥克兰市以非常紧迫的时间表交付给丹麦邮政银行。

我在 386BSD 和 FreeBSD 上认识的 Julian Elischer 在那里(TRW Financial Services)工作，他通过推荐“DGB”项目给我而获得了推荐奖金。这可能使我成为第二个因为 FreeBSD 而找到工作的人，Jordan 是第一个，他在 Walnut Creek CD-ROM 工作。

我成功获得了这份工作，并搬到了加利福尼亚州，在那里我恰好安顿在普莱森特希尔的一间公寓，距离 Jordan 和 Walnut Creek CDROM 只有几英里远。这一便利很快就派上了用场。

名义上，我是在美国接受培训，但很快我发现自己负责在五台性能强大的并配备了大量磁盘的 Sun Sparc 1000 服务器上安装 Solaris 和第三方软件；同时负责配置几个 Cisco 7010 路由器、Fore Systems 制造的 ATM 交换机，以及管理数十条连接着 135 台运行着 NT 系统的主机。

USL-BSD 的诉讼在 1994 年 2 月庭外和解，附有 3 个月的宽限期，该宽限期同样适用于 FreeBSD 1.0 版本。

因此，我们不得不重新开始，并在我们能够获得“4.4 BSD(Lite)”版本的代码后尽快展开工作。然后，我们不得不尽力重新整合 FreeBSD 1 中“未受限制”的部分，以实现与 FreeBSD 1.0 相同的功能。

我记得 Rod Grimes 完成了大部分工作，他在 1994 年 5 月份为我们准备好了新的 CVS 代码库。

不久之后，我加入了核心团队，并几乎立即被任命为 2.0 版本的发布工程师，与 Jordan 一起担任这个职务。

在 FreeBSD 中，releng@是指定的负责将火车驶上坚实地面的驾驶员，而 Jordan 和我设立了一个先例，该工作几乎具有无限的权限。

作为发布工程师，我们需要编写构建版本的代码，将版本打包成与媒体无关的分发文件，编写启动安装程序的代码，安装程序本身，对硬盘进行分区的代码，从各种媒体和网络服务中提取分发文件的代码，以及进行启动引导块处理的神奇代码。

但这些工作我们只需要做一次。与之相比，后续的发布变得轻而易举，直到十多年后，有人终于受够了并编写了 bsdinstall 工具。

Jordan 几乎编写了 sysinstall 程序的全部内容，它实现了我们的设计目标，即事先询问所有问题，而不是像 Windows 或 Solaris 那样，在需要答案时才问，这让每个人都很恼火。如果你在安装过程中必须一直监督，那从 CD-ROM 或磁带安装的意义在哪里呢？

然而，磁盘分区编辑器的编写任务却落到了我身上，导致一些人认为我应该被禁止再编写用户界面代码，如果有必要的话。

---

在 FreeBSD 中，releng@是指定的负责将火车驶上坚实地面的驾驶员，而 Jordan 和我设立了一个先例，该工作几乎具有无限的权限。

**参观 Releng 邮轮的"make world"吧！**

一个耗费了很多时间和提交的工作是“消毒”发布过程，确保在构建发布时，不会将任何“不洁之物”从系统中泄漏到发布的分发文件中。

例如，相当多的 Makefile 可能会写成：
        `CFLAGS+= -I/sys`
而不是正确的写法：
        `CFLAGS+= -I${.CURDIR}/../../sys`

我们并没有完全实现可重复构建，因为在当时，将诸如**DATE**、**TIME**等瞬态信息编译进程序中是非常常见的。

在发布前三周，我将 GCC 2.6.1 引入了编译树，希望它能解决我们面临的众多问题，并在提交信息(fe7dee47009525)中明确表示这不容许讨论。GCC 2.6.2 在发布前一周发布，我匆忙进行了相关更改，于是 FreeBSD-2.0 发布时配备了一个非傻瓜式的 C 编译器。

ITAR（国际交付军事货物条例）的双用途出口法规禁止未经美国政府明确许可的情况下出口加密软件，而我们既没有时间、资金，也没有律师来考虑这一点，我们也不太可能获得出口源代码的许可，而且肯定无法在相关时间范围内获得。

这种监管制度显然是徒劳无益的：相关源代码的副本在南非，Mark Murray 通过匿名 FTP 使其可以在全世界（包括美国）自由获取和使用，因为美国只禁止出口，而并没有禁止进口。

就我个人而言，如果以后出现麻烦，我可能会选择无视 ITAR，指向源代码的全球可获取性，但是 Jordan 否决了这一点，因为尽管 FreeBSD 项目可能能够逃脱制裁，但 Walnut Creek 却不能在他们的 CD-ROM 中放入“违禁品”，如果他们想要在国外销售这些 CD-ROM，或者其他任何人也想这么做。

真正与此相关的是 crypt(3) 函数，它用于对密码进行加密以进行存储：在传统的 UNIX 中，该函数源自 DES 加密算法派生的，因此源代码绝对不能出口。

在 FreeBSD-1 时代，有人（我也不太确定是谁）编写了一个伪 crypt(3)函数，但它实际上并不好用，我们当然不会在 2.0 版本中发布它。由于没有其他人做出任何进展，所以就在 2.0 发布前的两周，我自己动手解决了这个问题。

---

他们生产了一批带有“1995 年 1 月”日期的新 CD-ROM，并在封面上为 Beastie（BSD 系统的吉祥物）穿上鲜艳的绿色运动鞋，从那时起，在 FreeBSD 的相关领域里 Beastie 就一直穿着这双鞋。

我之前有过尝试暴力破解传统 UNIX 密码的经验。在一份工作中，我发现公司的密码策略要求使用“字母和数字”，结果每个人都将车牌号作为密码。

在后来的一份工作中，我发现在位于 Ivrea 的 Olivetti 研究中心，很多人都将“Gloria”作为密码，而我在见过她后，我不怪他们这样做。

首先，我必须进行一些实验来看看我能做到什么程度。有多少代码假定了加密后密码的长度？长度最多可以到 80 个字符，并不成问题。可以使用哪些字符？几乎所有可见字符都可以使用。以此类推。

我让密码破解变得更加困难。我将“盐值”变得更大，以防止预先计算的字典攻击。我设计了算法，使其在 60 MHz 的 Pentium 处理器上需要更多的 CPU 时间，大约 34 毫秒，并且非常难以在 FPGAs 中实现。

为了标记这种新类型的密码，我选择了前缀$1$，并提交了带有适当可怕的提交信息的代码，然后继续进行下一个仍然过长的待办事项清单。

由此产生的代码，远非完美，被称为“md-5crypt”。因为它使用了已经在 RFC 中发布的 MD5 密码哈希函数，我们可以自由地出口它，包括源代码，而且我们从未遇到任何麻烦。

**BSD, BSD, BSD, GNU, BSD 和啤酒**

这是指：如果你忽略了许可证。

刚刚发生了一次周期性的 GNU vs. BSD 许可证的邮件风暴，为了平等地激怒所有人，我在提交 md5crypt 之前重新采用了我的旧“啤酒许可证”并将其加在了代码上。

我从未完全弄清楚为什么 md5crypt 被广泛复制，但确实如此。各种自由开源软件都在导入它，从 Perl 到 Apache 都有。在 GNU libc 中出现了一个去除序列号的版本。有人用 COBOL 编写了 md5crypt。还有人使用某种 C->JS 转换器编写了 JavaScript 版本，包括底层的 MD5 实现。

闭源软件也采用了它：Macromedia Flash 使用了 md5crypt。

Cisco 将 md5crypt 用于 IOS 操作系统，最终，甚至 Microsoft 也支持了\$1\$密码。

随着 md5crypt 的传播，它也出现在某些基于自由开源软件的公司在被收购之前进行“尽职调查”的过程中，一些律师们真的很难准确地弄清楚谁欠谁多少啤酒。

IBM 收购了 Whistle Communications，后者是一家制造基于 FreeBSD 的通信设备的公司。一位礼貌的 IBM 初级律师给我打电话，征求录音的许可，为打扰我而道歉，然后按照高级律师准备的非常严肃的法律问题与我进行了交流。我们都笑得很开心。

但我也收到了一封来自非常严肃的高级律师的挂号信，要求我“立刻”以书面形式回复，并加盖公证章，回答以下问题：我是否是 md5crypt 函数的唯一作者？我是否完全拥有版权，并能够提供许可证？我是否向$BigCorp 提供了啤酒许可证？如果是这样的话，是\$BigCorp 的具体部门，还是所有的\$BigCorp，或者是\$BigCorp 的分销商和代理商，还是\$BigCorp 产品的最终用户欠我啤酒？如果是这样，他们欠我多少啤酒，如何交付，以及谁将负责海关、税费等费用？欠我的啤酒数量与法律实体对软件使用和/或认为软件有用程度的关系是怎样的？
如果在相关司法管辖区域内没有啤酒，是否可以用其他酒精饮料替代？那些由于法律或其他原因无法向我寄送酒精饮料的人是否被禁止使用我的软件？他显然对此考虑过很多。我没有回复。

有趣的是，没有一个律师曾经问过我是否愿意通过标准自由开源软件许可证来使他们的工作更容易。我想这不是律师的作风？

最终，在 1994 年 11 月 22 日，我们按时发布了 FreeBSD 2.0 版本，正逢已经开始的互联网热潮。

Walnut Creek CD-ROM 计划在 1994 年 12 月发布 FreeBSD 2.0 版本的 CD-ROM，但错过了制作截止日期，将日期编辑为“1994 年 1 月”，然后制作了 CD-ROM，但随后发现了错误，又生产了一个带有“1995 年 1 月”日期的新批次 CD-ROM，并通过为 Beastie 配备鲜艳绿色运动鞋的封面来区别版本。从此，Beastie 在 FreeBSD 的相关领域中一直自豪地穿着这双鞋。

在发布了更多版本后，我因表现良好退出了 releng@职务，但 Jordan 则一直在继续担任发布工程师的角色，因为 FreeBSD 成为了 Walnut Creek 的一款重要产品。

**Critter 始终运行 FreeBSD-current 版本**

我继续利用我大量的业余时间进行 FreeBSD 的开发，并购买了一台 Gateway Handbook/486 笔记本电脑，这样我就可以在每天的“BART”通勤时间里更加高效地工作。那是我拥有的第一台“critter”笔记本电脑，也是我最喜欢的一台，但这个名字随后被传承给我之后拥有的十几台笔记本电脑，它们全部运行着 FreeBSD 的-current 版本。

在 1995 年，由于大家都在购买 PC，RAM 价格飙升，而我的“critter”只有 4MB 的 RAM，我注意到 FreeBSD 的 RAM 使用效率并不高。于是我深入研究，并发现在某个地方，“虚拟机内存”已经超越了 UNIX 系统。

传统的 malloc(3)实现可以在 K&R 的《旧约》中的 173-177 页找到，在没有虚拟内存的系统上，它可以很好地工作，因为整个进程都会被交换进出。但是在有虚拟内存的情况下，将空闲内存块的元数据保存在这些空闲内存块中意味着你可能会将许多未使用的页面调入内存，只为了释放更多的内存。

在 K&R 和 FreeBSD 之间的某个时期，有人通过不太在意重用空闲内存来“解决”了这个问题，直到内核在使用 sbrk(2)分配内存时拒绝再分配更多内存。

经过一些修改，部分灵感来自于一台非常老的计算机的文件系统，我设计了 phkmalloc，这是最早的真正按页组织的 malloc(3)实现之一，它让我的小型笔记本电脑明显变得更快。

这个设计的一个副作用是，它可以在创建任何混乱之前确定性地检测到一些错误的使用方式，尤其是双重释放和释放已修改指针。因此，我在 phkmalloc 中添加了几个调试标志，其中 A 表示在任何问题上调用 abort(2)，J 表示填充垃圾等等。

当我在“critter”上全局设置了 AJ 标志后，fsck(8)在下一次重启时会出现核心转储，随后在接下来的几年里，包括 inetd(8)，cvs(1)，getpwent(3)，BIND 和 OpenSSH 等太多的代码也发生了类似问题。

**1998 年大巫师会议**

在 1998 年的夏天，正值互联网泡沫的巅峰时期，多亏了赞助商在虚拟资金上的慷慨解囊，我们才得以在新奥尔良举行的 USENIX 年度技术大会上，第一次也可能是唯一一次以实际行动召集了整个核心团队。

Bruce Evans 来自澳大利亚，他曾经对这个想法持抵制态度，最终透露他事实上几乎完全失聪了，但他被说服前来，并得到至少一个韦恩斯·沃尔德（Wayne's World）风格的“我们不值得！我们不值得！”的问候。

Bruce 去世已有几年了，但在当时他已经是一个传奇。Andrew Tanenbaum 曾感谢他的 Minix386，Linus Torvalds 在最初的 Linux 公告中也向他致谢，而我们在 FreeBSD 中对他的无数代码审查、耐心解释和机智幽默永远感激不尽。愿他安息。

为了方便起见，我提出了一个关于 phkmalloc 的演讲，并被接受在“FreeNix”论坛上发表，显然这类似于小孩子在大人们严肃的 UNIX 会议上的小打小闹。

我患有慢性失去记忆名字和面孔的问题，但我一眼就认出了坐在前排的 Kirk McKusick 的标志性小胡子。他的照片多年来出现在 USENIX 和欧洲 UNIX 用户组杂志的众多文章中。

在那之前，我从未与 UNIX 的贵族们打过交道，我决定跳过第 20 张幻灯片，因为在那张幻灯片中 Kirk 的 fsck(8)程序在我发现的漏洞软件清单中名列榜首。但当我到达那一页时，肾上腺素让我如此兴奋，我直接一鼓作气把那一页读完了，直到第一排和会场其他地方的爽朗笑声将我停住。在会议现场，我的清单上有问题的代码不止有 Kirk 的。

所有我与之交谈的“受害者”都对此非常友好，他们认为展示这份名单是非常公平的，可以证明这是一个真实存在的问题。

Kirk 是一个特别酷的家伙，几年后我有机会与他一起在 UFS2 项目上工作。在 Kirk 和 Eric Allman 的“秘密巫师派对”上，我学到了 UNIX 贵族们知道如何狂欢。如果那天晚上的果汁被下毒，你们现在可能都在运行 Windows NT。

对我个人来说，去新奥尔良的旅行中最令我激动的是在自助早餐时遇到了 Dennis Ritchie，并且与他谈论 UNIX 的历史、时间计算、设备节点和网络问题，谈话持续了很长时间，以至于我们错过了早上的第一场演讲。得到他对我在 DEVFS 所做工作的认可对我来说意味着一切。

---

我希望有一天，会有一本由有能力的历史学家撰写的关于 FreeBSD 历史的权威性书籍。

**磁盘、分区、缓冲区和 VNODEs**

sysinstall 磁盘编辑器向我展示了文件系统和存储设备之间的接口相当混乱。例如，分区功能是在各个设备驱动程序中实现的。硬盘大多数情况下可以实现分区，但 RAM 磁盘、软盘和光盘等设备则不行。

如果您查看使用虚拟内存之前的 UNIX 系统，一切都是井然有序的，但在上面添加虚拟内存的方式让我创造了一个稍微有些贬义的术语“phd-ware”。一堆概念验证源代码就像在日出前将抛到寺院门前的孤儿一样。

在我开始的时候，存储领域有三个 phdware 实例：Heidemann 的可扩展 VNODEs，Seltzer 的 LFS 以及 Kirk 的 FFS/UFS。值得称赞的是，Kirk 后来回来了，并一直维护着 FFS/UFS。

在工作中，我纠缠着一个反常的磁盘布局。Veritas Volume Manager 强制我们每个磁盘制作 9 个分区（为了性能），然后将每对分区在不同磁盘上进行镜像（为了可靠性），最后将镜像的分区串联起来，形成了十个用于图像存储的文件系统分区。

如果我们可以首先进行磁盘的成对镜像，然后在这些镜像上划分分区，最后再进行分区串联，那么设置、管理和运维都会更加合理和容易。但是 Veritas 无法实现这样的操作。

假设更糟糕的情况下，我们有五台服务器，但只有四个磁盘柜，每个 SCSI 总线上有三块硬盘，但每个电源供应器只有两个 SCSI 总线。

在投入生产一年后，这个特定的 Veritas 架构“限制”成为导致五十万份财务交易文件图像丢失的根本原因。

幸运的是，这些图像还能重新扫描获取，但生产延误了将近一个星期。

这个事件比其他任何事情都让我渴望并设计了一个类似 Lego™ 风格的卷管理系统，不受任何愚蠢的限制。

但在我实施它之前，还有很多问题要解决，我成功地解决了这些问题，以至于“Danish axes”在核心团队中成为了一个长期的笑话。我喜欢把它看作是在“技术债务”变得流行之前清理技术债务。

我不能自称是按照一份完美的内核架构蓝图工作，更多的是一连串的“嗯，这样可能更聪明……”在很多情况下，我多次绕回内核的相同部分，实现另一组改进。

最后，GEOM 的核心部分在 2002 年在 DARPA 合同下在“CHATS”研究项目下开发，得益于年轻而雄心勃勃的 Robert Watson，他还与我共同撰写了有关 Jails 的论文。

后来，我再次在 2004 年下半年发现我的一人公司的日程空闲，决定尝试“社区资助”是否能够支撑我对 FreeBSD 的热爱。这超出了所有的期望，共有 530 次提交，清理了 ttys、linedisciplines、filedescriptors、VNODE-switch、VFS-switch 等等。

**内核的黑历史**

但是我在项目中还有另一个角色——我是第一批核心团队成员之一。

核心团队是该项目的名义管理委员会，但是没有对这些事物进行正式定义，我们没有章程，而且我们面临着三个巨大的内部障碍：

首先，我们来自不同的国家，文化差异非常明显。

在丹麦，我们开玩笑说，如果三个人等车等了 5 分钟，等公交车到了的时候他们已经组成了一个协会。而在其他国家，组建协会和组织是非常正式的活动，需要涉及律师、有印章的公证人、文件费用，甚至政府的预先批准。

其次，更糟糕的是，我们都是非常优秀的编码人员，这个人群通常以人际交往能力不强而闻名，而且我们都习惯于是房间里、建筑物里，甚至可能是邮政编码里最聪明的人。相信我，猞猁不擅长领导猫群。

第三，FreeBSD 核心团队是在通常原则“他干得好，拉他进来！”的基础上有机地发展起来的。但出于某种误导的尊重，即使他们已经离开多年，我们也从未让任何人退出。

最终，我们的核心团队达到了 18 名名义成员的顶峰，其中不到一半能够在需要投票时投下票，这经常导致输掉的一方争辩称没有达到法定人数。

由于这三个障碍，核心团队有很多工作要做，最重要的是分发提交权限，这是我们不应该轻视的事情。在一些不愉快的情况下，当强烈鼓励不合适或行为不端的人自愿归还提交权限失败时，就需要再次收回提交权限。

**如果你能保持它...**

由于一个特定的“人员问题”，core@的合法性危机达到了临界点，尽管提出了各种次要的补丁，但它们不会也无法解决根本性的问题：core@不愿意也无法做出不愉快但必要的决定。

最终，我提议让提交者们选举一个新的 core 团队，但这个想法对那些来自历史上选举不够公正的国家的人并不吸引人。他们认为这只会导致“政治”，“竞选”和“腐败”。

我猜想，最终让他们相信我的想法的理由是，我说如果举行选举，我不会成为候选人，但除此之外，我会在光天化日之下，在大庭广众之下，坚守我的核心立场，直到最后，或者直到他们把我的核心立场撬走。

对我来说，履行这个承诺并不难，因为我的个人生活非常艰难。我（当时的）妻子的精神疾病迫使我成为个体经营者，从家里工作，照顾我们的两个幼儿。

不管怎样，最终在我们同意为 FreeBSD 制定了一套非常简单的规则后，我终于得到了我的选举机会。

在 2000 年秋季的 BSD 大会上，第一届核心团队选举结果正式公布，选出了一个能干的 9 人核心团队，而“新闻稿”邮件中写着：

> 离任的核心团队成员 Poul-Henning Kamp 说：“在过去的 8 年里，我为我们在核心团队所取得的成就感到非常自豪。新的核心团队由投票选出，意味着未来项目将对变革更加响应灵活。”在过去的 24 年里，投票选举核心团队的工作对于投票者来说是非常认真的。我们现在已经进行了 12 次合法选举出来的核心团队，其中没有一个比“core.0”时期差，许多核心团队显然都更加出色。对我来说，民主是我对 FreeBSD 最重要的贡献。

**到此为止...**

在第二次尝试中，我想我成功地避免了 "四个约克郡人 "的版权索赔，但我仍然觉得这从根本上是错误的，因为 FreeBSD 的历史远比我在上面随意挑选的自恋式回忆所能展现的要丰富得多。

我所提到的人名数量是我所能做到的最少的，因为实际上有太多人值得一提，他们无法在这篇篇幅有限的文章中涵盖。

我希望有一天，会有一本关于 FreeBSD 历史的权威著作，由一位称职的历史学家撰写，书中没有或很少有自己立场的牵扯，每个为 FreeBSD 作出贡献的人，包括我在内，都能得到公平的关注。

在这之前，向所有 FreeBSD 的朋友致敬，无论是过去的、现在的还是将来的：谢谢大家，请继续保持热情！

/phk

---

**POUL HENNING-KAMP**，即phk@FreeBSD.org，他的笔记本电脑叫做“critter”，已经运行了 FreeBSD-current 接近 30 年。在最近十多年里，平均每 18 小时他都会提交代码到 FreeBSD 源代码库，只有在 Norvegian Newspaper 遇到 HTTP 性能问题时他才会停止提交代码。