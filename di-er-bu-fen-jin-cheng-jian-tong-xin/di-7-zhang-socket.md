# 第 7 章 套接字

## 7.1. 概述

BSD socket 将进程间通信提升到了一个新的层次。通信的进程不再必须运行在同一台机器上。它们*可以*运行在同一台机器上，但不再是必须的。

不仅如此，这些进程也不需要运行在相同的操作系统下。多亏了 BSD socket，你的 FreeBSD 软件可以顺利地与一台 Macintosh® 上运行的程序协作，与一台 Sun™ 工作站上的程序协作，甚至与一台运行 Windows® 2000 的程序协作，所有这些通过基于以太网的局域网连接在一起。

而你的软件同样可以与运行在另一栋楼、另一个大陆、甚至潜艇或航天飞机中的进程协作。

它也可以与不是计算机（至少不是严格意义上的计算机）中的进程协作，比如打印机、数码相机、医疗设备等设备中的进程。几乎任何能进行数字通信的东西。

## 7.2. 网络与多样性

我们已经暗示过网络的*多样性*。许多不同的系统必须彼此通信。而它们必须讲相同的语言。它们还必须*以相同的方式理解*这种语言。

人们常常以为*肢体语言*是通用的。但事实并非如此。在我十几岁的时候，我父亲带我去了保加利亚。我们在索非亚的一座公园里坐在一张桌边，一个小贩走近我们，试图卖我们一些烤杏仁。

那时我还没怎么学过保加利亚语，于是，我没有说“不”，而是左右摇头，这是“*不*”这个意思的“通用”肢体语言。结果小贩立刻开始给我们装杏仁。

这时我才记起有人告诉过我，在保加利亚，摇头表示“*是*”。我赶紧改为上下点头。小贩注意到了，拿起杏仁，离开了。对一个不知情的旁观者来说，我的肢体语言没有变化：我继续摇头点头。变的是*肢体语言的含义*。起初，小贩和我对相同的语言作出了完全不同的解释。我必须调整自己对这种语言的解释，才能让小贩理解。

计算机之间也是如此：相同的符号可能具有不同，甚至完全相反的含义。因此，两个计算机要能互相理解，不仅要使用相同的*语言*，还要以相同的*方式解释*这种语言。

## 7.3. 协议

尽管各种编程语言通常具有复杂的语法，并使用许多多字母的保留字（这使得它们对程序员更易于理解），数据通信的语言通常非常简洁。它们不使用多字节的单词，而是经常使用单个的*比特*。这样做有一个非常令人信服的理由：虽然数据在计算机内部的传输速度接近光速，但在两台计算机之间的传输速度往往要慢得多。

由于数据通信中使用的语言非常简洁，我们通常将其称为*协议*，而不是语言。

当数据从一台计算机传输到另一台计算机时，它总是使用多个协议。这些协议是*分层的*。数据可以比作洋葱的内部：你需要剥去几层“皮”才能获得数据。用一张图片来说明这一点会更好：

![layers](https://docs.freebsd.org/images/books/developers-handbook/layers.png)

**图 1. 协议层**

在这个例子中，我们正在尝试从一个通过以太网连接的网页中获取一张图片。

图片由原始数据组成，原始数据只是一个 RGB 值的序列，我们的软件可以处理这些值，即将其转换为图像并显示在显示器上。

遗憾的是，我们的软件无法知道这些原始数据是如何组织的：它是 RGB 值的序列，还是灰度强度的序列，或者是 CMYK 编码的颜色？数据是用 8 位量表示，还是 16 位，或者是 4 位？图片由多少行和列组成？某些像素是否需要透明？

我想你已经明白了……

为了告诉我们的软件如何处理这些原始数据，它被编码为 PNG 文件。它也可以是 GIF 文件，或者 JPEG 文件，但它是 PNG。

而 PNG 就是一个协议。

在这个时候，我听到有些人喊道：“不，它不是！它是文件格式！”

当然，它是文件格式。但从数据通信的角度来看，文件格式就是协议：文件结构是一个*语言*，它简洁到极点，告诉我们的*进程*数据是如何组织的。因此，它是一个*协议*。

然而，如果我们收到的只是 PNG 文件，我们的软件将面临一个严重的问题：它怎么知道数据表示的是图像，而不是某些文本，或者是音频，或者其他什么？其次，怎么知道图像是 PNG 格式而不是 GIF、JPEG 或其他图像格式？

为了获得这些信息，我们使用了另一个协议：HTTP。这个协议可以准确地告诉我们，数据表示的是图像，并且它使用的是 PNG 协议。它还可以告诉我们其他信息，但我们现在还是集中在协议层次上。

所以，现在我们收到了一个包裹在 PNG 协议中的数据，再包裹在 HTTP 协议中。那我们是如何从服务器获得它的呢？

是通过 Ethernet 上的 TCP/IP 来的，实际上那是三个协议。为了更容易解释接下来的内容，我现在将重点讲解 Ethernet。

以太网是一个有趣的局域网（LAN）计算机连接系统。每台计算机都有一个*网络接口卡*（NIC），该卡具有一个唯一的 48 位 ID，称为*地址*。世界上没有两台 Ethernet NIC 拥有相同的地址。

这些 NIC 彼此连接。当一台计算机想要与同一 Ethernet LAN 中的另一台计算机通信时，它会通过网络发送消息。每个 NIC 都能看到这条消息。但作为以太网*协议*的一部分，数据中包含了目标 NIC 的地址（以及其他信息）。因此，只有其中一块网络接口卡会注意到这条消息，其他的都会忽略它。

但并不是所有的计算机都连接在同一个网络上。仅仅因为我们通过以太网接收到数据，并不意味着它来自我们自己的局域网。它可能来自其他通过 Internet 连接的网络（可能并非基于以太网的网络）。

所有数据通过互联网传输都使用 IP，即*互联网协议*。它的基本作用是告诉我们数据来自哪里，应该到达哪里。它不*保证*我们会收到数据，只是保证如果我们收到了数据，我们会知道它来自哪里。

即使我们收到了数据，IP 也不保证我们会按发送方发送的顺序接收到不同的数据块。例如，我们可能会先收到图像的中央部分，然后才收到左上角，或者右下角。

正是 TCP（*传输控制协议*）要求发送方重新发送任何丢失的数据，并将所有数据按正确的顺序排列。

总之，从一台计算机传输到另一台计算机，告诉它一张图像是什么样子的，竟然用了*五个*不同的协议。我们收到了包裹在 PNG 协议中的数据，再包裹在 HTTP 协议中，再包裹在 TCP 协议中，再包裹在 IP 协议中，最后包裹在以太网协议中。

哦，顺便说一句，可能还有许多其他协议参与了这个过程。例如，如果我们的局域网通过拨号连接到 Internet，那么它就用了 PPP 协议，通过调制解调器传输，调制解调器又使用了一个（或多个）不同的调制解调协议，等等，等等……

作为开发人员，你现在应该会问：“我该如何处理这一切？”

幸运的是，你并不需要处理所有这些内容。你*需要*处理其中的一部分，但不是全部。具体来说，你不必担心物理连接（在我们这个例子中是 Ethernet 和可能的 PPP 等）。你也不必处理互联网协议或传输控制协议。

换句话说，你不需要做任何事情来接收来自另一台计算机的数据。嗯，你确实需要*请求*它，但这几乎就像打开一个文件一样简单。

一旦你接收到数据，就由你来决定如何处理它。在我们的例子中，你需要理解 HTTP 协议和 PNG 文件结构。

举个比喻，所有的网络协议变成了一个灰色地带：这不仅仅是因为我们不理解它是如何工作的，而是因为我们不再关注它。套接字接口为我们处理了这一灰色地带：

![slayers](https://docs.freebsd.org/images/books/developers-handbook/slayers.png)

**图 2. 套接字覆盖的协议层**

我们只需要理解任何告诉我们如何*解释数据*的协议，而不是如何*从另一个进程接收*数据，或如何*发送*数据到另一个进程。


## 7.4. 套接字模型

BSD 套接字是建立在基本 UNIX® 模型之上的：*一切皆文件*。在我们的例子中，套接字可以让我们接收一个*HTTP 文件*，可以这么说。然后，剩下的工作就是从中提取出*PNG 文件*。

由于互联网的复杂性，我们不能简单地使用 `open` 系统调用或 `open()` C 函数。相反，我们需要采取多个步骤来“打开”一个套接字。

然而，一旦我们完成这些步骤，我们就可以开始像处理任何*文件描述符*一样处理*套接字*：我们可以从中`read`、`write`、`pipe`，最后`close`它。

## 7.5. 必要的套接字函数

虽然 FreeBSD 提供了多种函数来操作套接字，但我们只需要四个来“打开”一个套接字。在某些情况下，我们甚至只需要两个。

### 7.5.1. 客户端与服务器的区别

通常，套接字数据通信的一端是*服务器*，另一端是*客户端*。

#### 7.5.1.1. 共同的元素

##### 7.5.1.1.1. `socket`

客户端和服务器都使用的一个函数是 [socket(2)](https://man.freebsd.org/cgi/man.cgi?query=socket&sektion=2&format=html)。它的声明如下：

```
int socket(int domain, int type, int protocol);
```

返回值与 `open` 相同，都是整数。FreeBSD 从与文件句柄相同的池中分配它的值。这使得套接字可以像文件一样被处理。

`domain` 参数告诉系统你希望使用的*协议族*。有许多协议族，其中一些是厂商特定的，其他的则是常见的。它们在 **sys/socket.h** 中声明。

对于 UDP、TCP 和其他 Internet 协议（IPv4），使用 `PF_INET`。

`type` 参数有五个定义的值，也在 **sys/socket.h** 中声明。所有这些值都以 “SOCK\_” 开头。最常见的是 `SOCK_STREAM`，它告诉系统你请求一个*可靠的流式传输服务*（与 `PF_INET` 一起使用时是 TCP）。

如果你请求的是 `SOCK_DGRAM`，你将请求一个*无连接的数据报传输服务*（在我们的例子中是 UDP）。

如果你想控制底层协议（如 IP），甚至是网络接口（如以太网），你需要指定 `SOCK_RAW`。

最后，`protocol` 参数取决于前两个参数，并非总是有意义。在这种情况下，可以将其值设置为 `0`。

>**注意**
>
>未连接的套接字在 `socket` 函数中，我们并没有指定该套接字要连接到哪个其他系统。我们新创建的套接字仍然是*未连接的*。这是故意的：用电话的类比来说，我们刚刚将调制解调器接入电话线。我们既没有告诉调制解调器拨打电话，也没有告诉它在电话响起时接听。

##### 7.5.1.1.2. `sockaddr`

各种套接字系列的函数期望得到（或使用 C 术语的指针）内存中一小块区域的地址。这些 C 声明在 **sys/socket.h** 中将其称为 `struct sockaddr`。这个结构在同一个文件中声明：

```
/*
 * 内核用于存储大多数地址的结构
 */
struct sockaddr {
	unsigned char	sa_len;		/* 总长度 */
	sa_family_t	sa_family;	/* 地址族 */
	char		sa_data[14];	/* 实际上更长；地址值 */
};
#define	SOCK_MAXADDRLEN	255		/* 最长的可能地址 */
```

请注意 `sa_data` 字段的*模糊性*，它仅声明为 `14` 字节的数组，注释提示它可能包含超过 `14` 字节的数据。

这种模糊性是故意的。套接字是一个非常强大的接口。虽然大多数人可能认为它不过是一个 Internet 接口——并且大多数应用程序现在可能就是这样使用它——套接字几乎可以用于任何形式的进程间通信，其中互联网（或更确切地说是 IP）只是其中之一。

**sys/socket.h** 将套接字所处理的各种协议称为*地址族*，并在 `sockaddr` 定义之前列出它们：

```
/*
 * 地址族
 */
#define	AF_UNSPEC	0		/* 未指定 */
#define	AF_LOCAL	1		/* 本地主机（管道、端口） */
#define	AF_UNIX		AF_LOCAL	/* 向后兼容 */
#define	AF_INET		2		/* 内联网：UDP、TCP 等 */
#define	AF_IMPLINK	3		/* ARPANET IMP 地址 */
#define	AF_PUP		4		/* PUP 协议：如 BSP */
#define	AF_CHAOS	5		/* MIT CHAOS 协议 */
#define	AF_NS		6		/* XEROX NS 协议 */
#define	AF_ISO		7		/* ISO 协议 */
#define	AF_OSI		AF_ISO
#define	AF_ECMA		8		/* 欧洲计算机制造商 */
#define	AF_DATAKIT	9		/* DataKit 协议 */
#define	AF_CCITT	10		/* CCITT 协议，X.25 等 */
#define	AF_SNA		11		/* IBM SNA */
#define AF_DECnet	12		/* DECnet */
#define AF_DLI		13		/* DEC 直接数据链路接口 */
#define AF_LAT		14		/* LAT */
#define	AF_HYLINK	15		/* NSC 超级通道 */
#define	AF_APPLETALK	16		/* Apple Talk */
#define	AF_ROUTE	17		/* 内部路由协议 */
#define	AF_LINK		18		/* 链路层接口 */
#define	pseudo_AF_XTP	19		/* eXpress Transfer 协议（无 AF） */
#define	AF_COIP		20		/* 面向连接的 IP，亦称 ST II */
#define	AF_CNT		21		/* 计算机网络技术 */
#define pseudo_AF_RTIP	22		/* 帮助识别 RTIP 包 */
#define	AF_IPX		23		/* Novell Internet 协议 */
#define	AF_SIP		24		/* 简单互联网协议 */
#define	pseudo_AF_PIP	25		/* 帮助识别 PIP 包 */
#define	AF_ISDN		26		/* 综合服务数字网络 */
#define	AF_E164		AF_ISDN		/* CCITT E.164 推荐 */
#define	pseudo_AF_KEY	27		/* 内部密钥管理功能 */
#define	AF_INET6	28		/* IPv6 */
#define	AF_NATM		29		/* 原生 ATM 访问 */
#define	AF_ATM		30		/* ATM */
#define pseudo_AF_HDRCMPLT 31		/* BPF 用于不重写头部
					 * 在接口输出例程中
					 */
#define	AF_NETGRAPH	32		/* Netgraph 套接字 */
#define	AF_SLOW		33		/* 802.3ad 慢协议 */
#define	AF_SCLUSTER	34		/* Sitara 集群协议 */
#define	AF_ARP		35
#define	AF_BLUETOOTH	36		/* 蓝牙套接字 */
#define	AF_MAX		37
```

用于 IP 的是 AF\_INET，它是常量 `2` 的符号表示。

正是 `sockaddr` 中列出的 *地址族* 决定了如何使用 `sa_data` 字段中那些模糊命名的字节。

具体来说，当*地址族*为 AF\_INET 时，我们可以在需要 `sockaddr` 的地方使用 **netinet/in.h** 中的 `struct sockaddr_in`：

```
/*
 * 套接字地址，互联网风格。
 */
struct sockaddr_in {
	uint8_t		sin_len;
	sa_family_t	sin_family;
	in_port_t	sin_port;
	struct	in_addr sin_addr;
	char	sin_zero[8];
};
```

我们可以通过以下方式可视化它的组织结构：

![sain](https://docs.freebsd.org/images/books/developers-handbook/sain.png)

**图 3. `sockaddr_in` 结构**

三个重要的字段是 `sin_family`，它是结构的第 1 字节；`sin_port`，它是 16 位的值，存储在第 2 和第 3 字节中；以及 `sin_addr`，它是一个 32 位的 IP 地址整数表示，存储在第 4 至第 7 字节中。

现在，让我们尝试填充它。假设我们正在为 *daytime* 协议编写客户端，该协议简单地规定其服务器会将当前的日期和时间的文本字符串写入端口 13。我们希望使用 TCP/IP，因此我们需要在地址族字段中指定 `AF_INET`。`AF_INET` 被定义为 `2`。让我们使用 IP 地址 `192.43.244.18`，这是美国联邦政府的时间服务器（`time.nist.gov`）。

![sainfill](https://docs.freebsd.org/images/books/developers-handbook/sainfill.png)

**图 4. `sockaddr_in` 的具体示例**

顺便提一下，`sin_addr` 字段被声明为 `struct in_addr` 类型，它在 **netinet/in.h** 中定义：

```
/*
 * Internet 地址（出于历史原因的结构）
 */
struct in_addr {
	in_addr_t s_addr;
};
```

此外，`in_addr_t` 是一个 32 位的整数。

`192.43.244.18` 只是通过列出其所有 8 位字节（从*最重要*的字节开始）来表示一个 32 位整数的便捷表示法。

到目前为止，我们已经将 `sockaddr` 视为一种抽象。我们的计算机并不会将 `short` 整数存储为单一的 16 位实体，而是作为一系列的 2 个字节。类似地，它将 32 位整数存储为一系列 4 个字节。

假设我们编写了如下代码：

```
sa.sin_family      = AF_INET;
sa.sin_port        = 13;
sa.sin_addr.s_addr = (((((192 << 8) | 43) << 8) | 244) << 8) | 18;
```

结果会是什么样子呢？

当然，这取决于具体的计算机系统。在一个奔腾或其他 x86 系统上，它会像这样显示：

![sainlsb](https://docs.freebsd.org/images/books/developers-handbook/sainlsb.png)

**图 5. 在 Intel 系统上的 `sockaddr_in`**

在不同的系统上，可能会是这样的：

![sainmsb](https://docs.freebsd.org/images/books/developers-handbook/sainmsb.png)

**图 6. 在 MSB 系统上的 `sockaddr_in`**


在 PDP 上，它的表现可能又会不同。但上述两种方式是如今最常见的两种实现方式。

通常，为了编写可移植的代码，程序员会假装这些差异并不存在。而他们确实也能蒙混过关（除非他们在写汇编语言）。不过，当你在为 sockets 编程时，你就不能这么轻松地蒙混过关了。

为什么？

因为在与另一台计算机通信时，你通常不知道它是以 *最高有效字节*（MSB）优先还是 *最低有效字节*（LSB）优先来存储数据的。

你可能会想，*“那 sockets 难道不会替我处理这些吗？”*

不会。

这个答案可能一开始让你感到惊讶，但请记住，通用的 sockets 接口只理解 `sockaddr` 结构中的 `sa_len` 和 `sa_family` 字段。你不必担心这些字段的字节序（当然，在 FreeBSD 上 `sa_family` 反正只有 1 个字节，但许多其他 UNIX® 系统没有 `sa_len` 字段，并且使用 2 字节的 `sa_family` 字段，并且期望数据以该计算机的本地顺序存储）。

但对 sockets 而言，其余的数据就只是 `sa_data[14]`。根据 *地址族* 的不同，sockets 只是将这些数据转发到目标地。

事实上，当我们输入一个端口号时，是为了让另一台计算机知道我们请求的是什么服务。而当我们是服务器时，我们读取端口号，是为了知道对方期望我们提供什么服务。无论是哪种情况，sockets 只是将端口号当作数据转发。它不会对其进行任何解释。

同样，我们输入 IP 地址，是为了告诉网络上的所有中间设备数据应该被送往何处。sockets 依旧只是将其当作数据转发。

这就是为什么，我们（*程序员*，而不是 *sockets*）必须区分我们的计算机使用的字节序与用于发送给另一台计算机的标准字节序。

我们称我们的计算机使用的字节序为 *主机字节序*，简称 *主机序*。

而在 IP 上传送多字节数据有一个约定，就是以 *MSB 优先* 的方式传送。我们称这种顺序为 *网络字节序*，简称 *网络序*。

现在，如果我们将上面的代码编译为运行在 Intel 架构的计算机上，我们的 *主机字节序* 将会产生如下结果：

![sainlsb](https://docs.freebsd.org/images/books/developers-handbook/sainlsb.png)

**图 7. Intel 系统上的主机字节序**

但 *网络字节序* 要求我们以 MSB 优先的方式存储数据：

![sainmsb](https://docs.freebsd.org/images/books/developers-handbook/sainmsb.png)

**图 8. 网络字节序**

不幸的是，我们的 *主机序* 与 *网络序* 完全相反。

我们有几种方式可以应对这种情况。其中一种方法是在代码中 *反转* 这些值：

```c
sa.sin_family      = AF_INET;
sa.sin_port        = 13 << 8;
sa.sin_addr.s_addr = (((((18 << 8) | 244) << 8) | 43) << 8) | 192;
```

这样可以“骗过”我们的编译器，让它以 *网络字节序* 存储这些数据。在某些情况下，这确实是正确的做法（比如当你在写汇编语言的时候）。但在大多数情况下，这会引起问题。

假设你用 C 写了一个基于 sockets 的程序。你知道它会运行在奔腾上，于是你将所有常量反转后强行设为 *网络字节序*。一切正常。

直到有一天，你信赖的旧奔腾变成了锈迹斑斑的老古董。你换了一台新机器，它的 *主机序* 正好和 *网络序* 一致。你重新编译了你的所有软件。所有程序都运作良好，除了你那一个程序。

你已经忘了当初你曾经强行把所有常量写成了和 *主机序* 相反的顺序。你开始拔头发，呼唤你听过和没听过的所有神的名字，用海绵棒猛击显示器，进行各种传统的“调试祭祀仪式”，试图搞清楚为什么一个一直工作良好的程序突然就失效了。

最终，你搞清楚了问题的根源，骂了几句脏话，然后开始重写你的代码。

幸运的是，你不是第一个遇到这个问题的人。早有人已经创建了 [htons(3)](https://man.freebsd.org/cgi/man.cgi?query=htons&sektion=3&format=html) 和 [htonl(3)](https://man.freebsd.org/cgi/man.cgi?query=htonl&sektion=3&format=html) 这两个 C 函数，分别用于将 `short` 和 `long` 从 *主机字节序* 转换为 *网络字节序*；还有 [ntohs(3)](https://man.freebsd.org/cgi/man.cgi?query=ntohs&sektion=3&format=html) 和 [ntohl(3)](https://man.freebsd.org/cgi/man.cgi?query=ntohl&sektion=3&format=html) 函数，用于反向转换。

在 *MSB 优先* 的系统上，这些函数不会进行任何操作。而在 *LSB 优先* 的系统上，它们会将值转换为正确的顺序。

所以，无论你的软件是在哪个系统上编译的，只要使用这些函数，你的数据最终就会以正确的顺序被传送出去。

#### 7.5.1.2. 客户端函数

通常，客户端负责发起与服务器的连接。客户端知道它要联系哪个服务器：它知道服务器的 IP 地址，也知道服务器所监听的 *端口*。这就像你拿起电话拨号（这个 *地址*），然后在有人接听后请求找负责“wingdings”的人（这个 *端口*）。

##### 7.5.1.2.1. `connect`

客户端创建了 socket 后，就需要将其连接到远程系统的一个特定端口。它会使用 [connect(2)](https://man.freebsd.org/cgi/man.cgi?query=connect&sektion=2&format=html)：

```c
int connect(int s, const struct sockaddr *name, socklen_t namelen);
```

参数 `s` 是 socket，也就是 `socket` 函数返回的值。`name` 是一个指向 `sockaddr` 的指针，我们前面已经详细讨论过该结构。最后，`namelen` 用于告知系统我们的 `sockaddr` 结构体的字节数。

如果 `connect` 调用成功，它会返回 `0`。否则返回 `-1`，并将错误代码存储在 `errno` 中。

`connect` 可能失败的原因有很多。例如，在尝试连接到某个互联网地址时，对方的 IP 地址可能根本不存在，或者对方主机宕机了，或者太忙，或者根本没有在指定端口监听任何服务。也有可能直接 *拒绝* 来自特定代码的任何请求。

##### 7.5.1.2.2. 我们的第一个客户端

现在我们已经掌握足够知识，来编写一个非常简单的客户端程序，它将从 `192.43.244.18` 获取当前时间并打印到 **stdout**。

```c
/*
 * daytime.c
 *
 * G. Adam Stanislav 编写
 */
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

int main() {
  int s, bytes;
  struct sockaddr_in sa;
  char buffer[BUFSIZ+1];

  if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
    perror("socket");      // 打开 socket 失败
    return 1;
  }

  memset(&sa, '\0', sizeof(sa));  // 清空结构体

  sa.sin_family = AF_INET;                  // 使用 IPv4
  sa.sin_port = htons(13);                 // 端口号 13（daytime 服务）
  sa.sin_addr.s_addr = htonl((((((192 << 8) | 43) << 8) | 244) << 8) | 18);  // IP 地址

  if (connect(s, (struct sockaddr *)&sa, sizeof sa) < 0) {
    perror("connect");     // 连接失败
    close(s);              // 关闭 socket
    return 2;
  }

  while ((bytes = read(s, buffer, BUFSIZ)) > 0)
    write(1, buffer, bytes);  // 输出到标准输出

  close(s);  // 关闭连接
  return 0;
}
```

现在请打开编辑器，输入上述内容，保存为 **daytime.c**，然后编译并运行它：

```sh
% cc -O3 -o daytime daytime.c
% ./daytime

52079 01-06-19 02:29:25 50 0 1 543.9 UTC(NIST) *
%
```

在本例中，日期是 2001 年 6 月 19 日，时间是 UTC 时间 02:29:25。当然，你运行程序时的输出会有所不同。

#### 7.5.1.3. 服务器函数

典型的服务器不会主动发起连接。它会等待客户端来调用它、请求服务。它不知道客户端什么时候会来，也不知道会有多少客户端会来。有时候它只是静静地坐在那里等待，而下一刻，可能就会突然被大量同时请求的客户端淹没。

sockets 接口提供了三个基本函数来处理这一情况。

##### 7.5.1.3.1. `bind`

端口就像电话线路上的分机：拨通一个号码后，还需要拨分机号才能联系到特定的人或部门。

IP 端口总共有 65535 个，但服务器通常只处理来自其中某一个端口的请求。这就像告诉电话总机我们正在工作，可以在某个特定分机上接电话。我们使用 [bind(2)](https://man.freebsd.org/cgi/man.cgi?query=bind&sektion=2&format=html) 告诉 sockets 我们要监听哪个端口。

```c
int bind(int s, const struct sockaddr *addr, socklen_t addrlen);
```

除了在 `addr` 中指定端口，服务器也可以包括它自己的 IP 地址。然而，它也可以使用符号常量 INADDR\_ANY 来表示将接收发往该端口的所有请求，而不管目标 IP 是什么。这个符号常量和其他几个类似常量都定义在 **netinet/in.h** 中：

```c
#define	INADDR_ANY		(u_int32_t)0x00000000
```

假设我们要写一个基于 TCP/IP 的 *daytime* 协议服务器。回忆一下，它使用端口 13。我们的 `sockaddr_in` 结构将如下所示：

![sainserv](https://docs.freebsd.org/images/books/developers-handbook/sainserv.png)

图 9. 示例服务器 sockaddr\_in

##### 7.5.1.3.2. `listen`

继续我们之前的电话比喻：当你告诉总机你在哪个分机接电话后，你就走进办公室，确保电话插好了，铃声打开了。此外，你还要确保电话支持“通话中等待”，以便即使你正在通话中也能听到新的来电。

服务器使用 [listen(2)](https://man.freebsd.org/cgi/man.cgi?query=listen&sektion=2&format=html) 函数来确保这一切。

```c
int listen(int s, int backlog);
```

这里的 `backlog` 参数告诉 sockets：在你还在处理上一个请求时，最多可以接受多少个挂起的请求。换句话说，它决定了待处理连接队列的最大长度。

##### 7.5.1.3.3. `accept`

电话铃响之后，你接起电话，就建立了与客户端的连接。该连接会一直保持，直到你或客户端挂断为止。

服务器使用 [accept(2)](https://man.freebsd.org/cgi/man.cgi?query=accept&sektion=2&format=html) 函数来接收连接：

```c
int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```

注意这次 `addrlen` 是一个指针。这是因为在这个调用中由 socket 来填写 `addr`，也就是 `sockaddr_in` 结构。

返回值是一个整数。实际上，`accept` 返回的是一个 *新 socket*。你将使用这个新 socket 来与客户端通信。

那旧的 socket 呢？它仍然监听更多的请求（还记得我们传给 `listen` 的 `backlog` 吗？），直到我们调用 `close`。

而新的 socket 仅用于通信。它是完全连接的，不能再传给 `listen` 去接受其他连接。

##### 7.5.1.3.4. 我们的第一个服务器

我们的第一个服务器比我们的第一个客户端要复杂一些：不仅使用了更多的 sockets 函数，而且我们需要将它写成一个守护进程（daemon）。

最好的方式是，在绑定端口之后创建一个 *子进程*。主进程随即退出，将控制权还给 shell（或调用它的其他程序）。

子进程调用 `listen`，然后进入一个无限循环，接受连接、提供服务、最终关闭该连接的 socket。

```c
/*
 * daytimed - 一个监听端口 13 的服务器
 *
 * G. Adam Stanislav 编写
 * 2001 年 6 月 19 日
 */
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define BACKLOG 4

int main() {
    int s, c;
    socklen_t b;
    struct sockaddr_in sa;
    time_t t;
    struct tm *tm;
    FILE *client;

    if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket");
        return 1;
    }

    memset(&sa, '\0', sizeof(sa));

    sa.sin_family = AF_INET;
    sa.sin_port   = htons(13);

    if (INADDR_ANY)
        sa.sin_addr.s_addr = htonl(INADDR_ANY);

    if (bind(s, (struct sockaddr *)&sa, sizeof sa) < 0) {
        perror("bind");
        return 2;
    }

    switch (fork()) {
        case -1:
            perror("fork");
            return 3;
        default:
            close(s);
            return 0;
        case 0:
            break;
    }

    listen(s, BACKLOG);

    for (;;) {
        b = sizeof sa;

        if ((c = accept(s, (struct sockaddr *)&sa, &b)) < 0) {
            perror("daytimed accept");
            return 4;
        }

        if ((client = fdopen(c, "w")) == NULL) {
            perror("daytimed fdopen");
            return 5;
        }

        if ((t = time(NULL)) < 0) {
            perror("daytimed time");
            return 6;
        }

        tm = gmtime(&t);
        fprintf(client, "%.4i-%.2i-%.2iT%.2i:%.2i:%.2iZ\n",
            tm->tm_year + 1900,
            tm->tm_mon + 1,
            tm->tm_mday,
            tm->tm_hour,
            tm->tm_min,
            tm->tm_sec);

        fclose(client);
    }
}
```

我们首先创建一个 socket。然后填写 `sa` 中的 `sockaddr_in` 结构体。注意对 INADDR\_ANY 的条件使用：

```c
if (INADDR_ANY)
        sa.sin_addr.s_addr = htonl(INADDR_ANY);
```

其值为 `0`。由于我们刚刚对整个结构体使用了 `bzero`，再次将其设为 `0` 是多余的。但如果我们将代码移植到某个 `INADDR_ANY` 也许不是零的系统上，就必须显式地将它赋值给 `sa.sin_addr.s_addr`。多数现代 C 编译器足够聪明，它们会发现 `INADDR_ANY` 是一个常量，只要它的值是零，就会自动优化掉整个条件语句。

成功调用 `bind` 之后，我们就准备好变成一个 *守护进程*（daemon）：我们使用 `fork` 创建一个子进程。在父进程和子进程中，变量 `s` 都是我们的 socket。父进程不再需要它，于是它调用 `close`，然后返回 `0`，告知其父进程它已经成功终止。

与此同时，子进程继续在后台运行。它调用 `listen`，将 `backlog` 设置为 `4`。这个值不需要太大，因为 *daytime* 并不是一个有很多客户端频繁请求的协议，而且每个请求也能被瞬间处理完毕。

最后，守护进程启动一个无限循环，按以下步骤操作：

1. 调用 `accept`。它会阻塞，直到有客户端连接。此时，它会获得一个新的 socket `c`，用于与这个特定客户端通信。
2. 使用 C 函数 `fdopen` 将 socket 从底层 *文件描述符* 转换为 C 风格的 `FILE` 指针，这样可以后续使用 `fprintf`。
3. 获取当前时间，并用 *ISO 8601* 格式打印到 `client` “文件”中。随后使用 `fclose` 关闭该文件，这也会自动关闭对应的 socket。

我们可以将这个模式 *泛化*，作为许多其他服务器的模板：

![serv](https://docs.freebsd.org/images/books/developers-handbook/serv.png)

**图 10. 顺序服务器（Sequential Server）**

这个流程图适用于 *顺序服务器*，即一次只能服务一个客户端的服务器，就像我们的 *daytime* 服务器一样。只有在客户端和服务器之间没有真正的“对话”时，这种方式才是可行的：一旦服务器检测到客户端连接，它便立即发送一些数据，然后关闭连接。整个过程可能只需几纳秒便完成。

这种流程图的优点在于，除了 `fork` 之后父进程退出之前那一瞬间，始终只有一个 *进程* 活跃：服务器不会占用太多内存和系统资源。

注意，我们在流程图中添加了 *初始化守护进程* 的步骤。虽然我们的例子中不需要初始化守护进程，但这是在程序流程中设置 `signal` 信号处理器、打开可能用到的文件等的良好位置。

几乎流程图中所有内容都可以原样用于许多不同的服务器中，唯独 *serve* 部分是个例外。我们可以把它当作一个 *“黑箱”*，即根据自己的服务器需求特别设计的部分，然后“插入”到其余部分中即可。

并非所有协议都如此简单。很多协议都需要从客户端接收请求、回复请求，然后再次接收同一客户端的新请求。因此，它们无法预先知道需要服务多长时间。这类服务器通常会为每个客户端创建一个新进程。在新进程服务其客户端的同时，守护进程仍可继续监听新的连接。

现在，请将上述源代码保存为 **daytimed.c**（按照惯例，守护进程的程序名以字母 `d` 结尾）。编译后尝试运行它：

```sh
% ./daytimed
bind: Permission denied
%
```

发生了什么？如你所知，*daytime* 协议使用的是端口 13。但所有小于 1024 的端口都是保留给超级用户的（否则任何人都可以伪装成一个守护进程，服务一个常见端口，从而造成安全漏洞）。

这次以超级用户身份再试一次：

```sh
# ./daytimed
#
```

什么…… 没有任何输出？我们再试一次：

```sh
# ./daytimed

bind: Address already in use
#
```

每个端口在同一时间只能被一个程序绑定。我们的第一次尝试实际上是成功的：它启动了子守护进程并静默返回。它仍在后台运行，并将一直运行，直到你终止它、它的系统调用失败，或者你重启系统。

很好，我们知道它在后台运行。但它真的 *工作* 吗？怎么知道它是一个正确的 *daytime* 服务器？很简单：

```sh
% telnet localhost 13

Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
2001-06-19T21:04:42Z
Connection closed by foreign host.
%
```

telnet 先尝试使用新的 IPv6，失败后改用 IPv4 并成功连接。守护进程正常运行。

如果你能通过 telnet 访问另一台 UNIX® 系统，也可以用它来远程测试服务器。我所用的计算机没有静态 IP 地址，因此我做了如下测试：

```sh
% who

whizkid          ttyp0   Jun 19 16:59   (216.127.220.143)
xxx              ttyp1   Jun 19 16:06   (xx.xx.xx.xx)
% telnet 216.127.220.143 13

Trying 216.127.220.143...
Connected to r47.bfm.org.
Escape character is '^]'.
2001-06-19T21:31:11Z
Connection closed by foreign host.
%
```

它确实工作了。那么用域名也行吗？

```sh
% telnet r47.bfm.org 13

Trying 216.127.220.143...
Connected to r47.bfm.org.
Escape character is '^]'.
2001-06-19T21:31:40Z
Connection closed by foreign host.
%
```

顺便说一句，telnet 在我们的守护进程关闭 socket 后打印 *Connection closed by foreign host* 消息，这证明我们代码中使用 `fclose(client);` 的做法确实起到了作用。

## 7.6. 辅助函数

FreeBSD 的 C 标准库包含许多用于 socket 编程的辅助函数。例如，在我们的示例客户端中，我们是将 `time.nist.gov` 的 IP 地址硬编码进程序的。但我们并不总是知道 IP 地址。即使知道，如果程序允许用户输入 IP 地址，甚至是域名，它也会更灵活。

### 7.6.1. `gethostbyname`

虽然没有办法将域名直接传递给任何 socket 函数，但 FreeBSD 的 C 库提供了 [gethostbyname(3)](https://man.freebsd.org/cgi/man.cgi?query=gethostbyname&sektion=3&format=html) 和 [gethostbyname2(3)](https://man.freebsd.org/cgi/man.cgi?query=gethostbyname2&sektion=3&format=html) 这两个函数，它们在 **netdb.h** 中声明。

```c
struct hostent * gethostbyname(const char *name);
struct hostent * gethostbyname2(const char *name, int af);
```

这两个函数都会返回一个指向 `hostent` 结构体的指针，该结构体中包含大量关于该域名的信息。对我们的用途来说，该结构体中的 `h_addr_list[0]` 字段指向正确地址的 `h_length` 字节，这些字节已经是 *网络字节序*。

这使得我们可以创建一个更加灵活——也更加实用——的 daytime 程序版本：

```c
/*
 * daytime.c
 *
 * 编写者：G. Adam Stanislav
 * 2001 年 6 月 19 日
 */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

int main(int argc, char *argv[]) {
  int s, bytes;
  struct sockaddr_in sa;
  struct hostent *he;
  char buf[BUFSIZ+1];
  char *host;

  if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
    perror("socket");
    return 1;
  }

  memset(&sa, '\0', sizeof(sa));

  sa.sin_family = AF_INET;
  sa.sin_port = htons(13);

  host = (argc > 1) ? argv[1] : "time.nist.gov";

  if ((he = gethostbyname(host)) == NULL) {
    herror(host);
    return 2;
  }

  memcpy(&sa.sin_addr, he->h_addr_list[0], he->h_length);

  if (connect(s, (struct sockaddr *)&sa, sizeof sa) < 0) {
    perror("connect");
    return 3;
  }

  while ((bytes = read(s, buf, BUFSIZ)) > 0)
    write(1, buf, bytes);

  close(s);
  return 0;
}
```

现在我们可以在命令行中输入域名（或 IP 地址，二者皆可），程序就会尝试连接该地址的 *daytime* 服务器。否则，它仍然会默认连接 `time.nist.gov`。不过即便是这种情况，我们也使用了 `gethostbyname`，而不是硬编码 `192.43.244.18`。这样一来，即使它将来更换了 IP 地址，我们依然可以找到它。

由于从本地服务器获取时间几乎不花什么时间，你可以连续运行两次 daytime：第一次从 `time.nist.gov` 获取时间，第二次从你自己的系统中获取。然后你就可以比较两者的结果，看看你的系统时钟有多精确：

```sh
% daytime ; daytime localhost

52080 01-06-20 04:02:33 50 0 0 390.2 UTC(NIST) *
2001-06-20T04:02:35Z
%
```

如你所见，我的系统时间比 NIST 时间快了两秒。

### 7.6.2. `getservbyname`

有时候你可能不确定某个服务使用的端口号。此时，[getservbyname(3)](https://man.freebsd.org/cgi/man.cgi?query=getservbyname&sektion=3&format=html) 函数就非常有用，它同样在 **netdb.h** 中声明：

```c
struct servent * getservbyname(const char *name, const char *proto);
```

`servent` 结构体包含 `s_port` 字段，其中保存了正确的端口号，且已经是 *网络字节序*。

如果我们事先不知道 *daytime* 服务使用的端口，可以这样获取：

```c
struct servent *se;
  ...
  if ((se = getservbyname("daytime", "tcp")) == NULL {
    fprintf(stderr, "Cannot determine which port to use.\n");
    return 7;
  }
  sa.sin_port = se->s_port;
```

通常你是知道端口号的。但如果你正在开发一个新的协议，可能会在一个非官方端口上测试它。某一天，你会为该协议和它的端口注册编号（哪怕只是写进你的 **/etc/services** 文件中，`getservbyname` 正是查这个文件）。在上述代码中你也可以不返回错误，而是临时指定一个端口号。一旦你将协议列入 **/etc/services**，你的软件就能自动找到对应端口，而无需重写代码。


## 7.7. 并发服务器

与顺序服务器不同，*并发服务器* 必须能够同时为多个客户端提供服务。例如，一个 *聊天服务器* 可能会为某个特定客户端服务几个小时——它不能等这个客户端断开了，才去服务下一个客户端。

这就要求我们对流程图进行重大改动：

![serv2](https://docs.freebsd.org/images/books/developers-handbook/serv2.png)

**图 11. 并发服务器**

我们将 *服务逻辑* 从 *守护进程* 中移到了独立的 *服务进程* 中。但由于每个子进程会继承所有已打开的文件（socket 被视为一种文件），新进程不仅会继承由 `accept` 返回的 *连接句柄*，也会继承由顶层进程最初创建的 *监听 socket*。

但 *服务进程* 并不需要这个监听 socket，因此应立即对它执行 `close` 操作。同样地，*守护进程* 也不再需要连接 socket，不仅应当关闭它，而且 *必须* 关闭——否则迟早会耗尽可用的 *文件描述符*。

当 *服务进程* 完成服务后，它应关闭连接 socket。此时不再返回 `accept`，而是直接退出。

在 UNIX® 中，进程实际上并不会真正 *退出*，而是会 *返回* 给它的父进程。通常父进程会调用 `wait` 来等待其子进程，并获取其返回值。但我们的 *守护进程* 不能就此停止并等待子进程完成。否则就违背了创建多个子进程的初衷。但如果它永远不调用 `wait`，子进程就会变成 \*僵尸进程（zombie）\*——虽然不再起作用，但仍残留在系统中。

因此，*守护进程* 需要在 *初始化阶段* 设置 *信号处理器*。至少要处理 SIGCHLD 信号，以便清除子进程的返回值，并释放它们占用的系统资源。

这也正是为什么流程图中多了一个不连接任何其他模块的 *处理信号* 方框。顺便说一句，许多服务器还会处理 SIGHUP 信号，并通常将其解释为超级用户发出的“重新读取配置文件”的信号。这样我们就可以更改设置，而不必杀掉并重启这些服务器。
