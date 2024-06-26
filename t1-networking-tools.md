# Networking Tools

## Overview

在这个指导手册中，我们会覆盖网络研究中经常使用的工具（更不用说在现实世界中调试使用了）。我们从netstat和tcpdump开始，通过查看一个活跃的TCP连接，并熟悉它们的输出结果，以此来简要介绍如何使用它们中的每一个。我们将会使用netstat来描述TCP状态机的每一个元素。

接下来，我们会学习一组基于查找的工具，dig和whois,它们提供了主机的DNS信息，并分别提供关于名字和网络的注册信息。

然后，我们会转换到标准网络测量工具，ping以及更加有趣的traceroute,并描述它们如何工作，以及它们能（不能）告诉你什么。

最后，对于你们中不知道perl的人，我将展示如何“将事情黏连起来”。具体方法是展示一个简单的perl脚本，他读取我们的trace结果，并计算到目的IP地址和AS网络号的字节数。

## 1. tcpdump and netstat

tcpdump是如何工作的？对于BSD演变的内核来说，伯克利包过滤器(BPF)是支持的。这个过滤器将设备驱动程序设置为混杂模式，从驱动接收模块接收所有发送和接收的报文。这些报文然后通过一个用户指定的过滤器，以便只有用户指定的感兴趣的报文才被送到用户进程中。通过常量方式实现内核对于小包读取的具体开销是多少，对于批量读取需要设置超时时间。

过滤器工作在内核中。这会限制从内核拷贝到用户态中的数据数量。

下面是在我的电脑上的一些输出结果，对应的命令是：tcpdump host -i eth0 aros.ron.lcs.mit.edu port ssh 

时间戳选项的具体含义是？比如，为什么不使用包发送的时间标记一个报文？(需要时钟同步)

每一行包含的信息有：时间戳，源和目的地址（地址和端口号），TCP flags, 段信息（开始，结束，大小），acks, 窗口大小，各种TCP的其他选项。其中“P”标记表示尽可能快的“push”接收的数据到应用程序。注意其中的三次握手。另外，注意断开连接的过程。

简要描述一下状态转换图，使用“rule set”的汇总信息，来介绍TCP是如何工作的。

为什么连接进入了time wait状态？注意这里还有名为“2MSL”(最大段生存时间)的状态。结束动作中执行主动关闭（active close）时不知道最后的ACK已经被接收了，所以它也不知道另一端关闭了当前的连接。仅仅因为FIN报文到达，我们不能断定连接已经被关闭了————因为报文可能乱序到达。

你能想到各种不同的TCP攻击么？Sequence number攻击是其中一种。SYN flood攻击是另外一种？你是如何防御SYN flood攻击的？一种方法是叫做“SYN Cookies”的方法，这是指发送方不会建立连接，直到当三次握手完成后（相对于进入当接收第一个SYN报文后进入的SYN接收状态）。在接下来的周课件中会讨论这个主题。

## 2. Lookup Tools: dig and whois

dig是一种DNS查询工具。他和nslookup很类似，但是功能更加强大。你可以查询各种类型的记录。其中最普遍（默认）的是“A”记录，这种一种记录DNS名字到IP地址的映射。

这是在我的机器上查询的一些样例输出。注意查询是针对“A”记录的。dig还可以返回一系列其他有用的信息。比如，它返回了对于lcs.mit.edu的“NS”记录。NS记录返回对于特定域名经过认证的DNS服务器。注意我还获得对于所有其他机器的“A”记录。为什么对于每个域都以点号结束。因为这可以告诉你名字是被完全授权的（试着在MIT的机器上分别查询ai和ai.）。

其中的数字“1800”描述了本地的DNS服务器应该缓存这个查询结果的时长。

当名字无法被解析的时候解释输出是什么样子。解释反向DNS查询，部分授权的域名等等。

你可能看到的其他事情还包括类似于CNAME的类型。人们经常需要对于一个机器起个别名，或者多个名字。这一般使用常用名，在DNS中对应于CNAME记录，下面是显示结果

人们还可以使用dig查询特定域名的邮件服务器地址。比如说有些人想要向feamster@lcs.mit.edu发送邮件，那么他们的邮件程序如何找到负责向lcs.mit.edu发送邮件的邮件服务器呢？这就使用一种名为“MX”的记录。我可以演示一下这种查询，比如输入dig lcs.mit.edu MX。那些数字是怎么回事？它们用于标表示相对于其他服务器的优先级。

你还可以使用dig做反向查询。通过使用dig -x最容易做这种查询。dig还可以被用于批量模式。通过查询man帮助手册获取更多信息。

whois也是一种允许你查询任何whois服务器的有用工具。有各种类型的whois服务器。一种是nic服务器，它可以告诉你指定的名字是谁注册的信息。在我的机器上默认的whois服务器是whois.cranic.net。这个whois服务器然后会触发到另一个whois服务器的查询，后者负责注册名字信息。下面显示一种输出

whois还可以被用于查询地址注册，包括ARIN，APNIC，RIPE。比如说，我想知道关于18.31.0.38的更多信息。

另外需要注意最数据库后一次被更新的日期，以及记录什么时候被最后一次更新，这可以给你一些关于信息准确性的想法。

whois 服务器除了被用于解析名字到管理域的映射，还存在帮助映射数字到管理域的 whois 服务器。特别地，这些数字(IP 地址和 AS 号码)被以下组织管理。

如果我想知道 MIT 关于地址或者 AS 号的一些事情，我可以问询 ARIN whois 服务器。ARIN 的 whois 服务器有很多友好的选项，比如你可以限制查询特定记录的次数等。比如说我想查询“AS 701”。

对超级网络的查询也得到了支持。APINIC 和 RIPE 以类似的方式工作。我们应该如何查找特定网络或者 IP 地址的自治系统，还有另一个 whois 数据库，成为 RADB，它保持的信息相对较新。请注意，这是 traceroute -A 如何将 IP 地址解析为 AS 的。

## 3. Measurement Tools : ping and traceroute

ping是一个简单的程序，它发送一个ICMP "Echo Request"消息给目标机器，然后等待ICMP "Echo Reply"消息。在告诉一台特定的机器是否可以响应的时候它特别有用(因此在网络中他一般默认都是打开状态)。Ping对于每个request/reply消息对还会显示一个RTT值，以及一个序列号。Ping还对丢包率有一个总体的描述。注意RTT并不总是完全精确的，因为ICMP对于许多路由器来说并一定走最快路径。其他有意思的选项还包括“f”（flood, 有时候需要root权限），“c”(发送特定数量的报文)， "s"(表示特定报文大小，有时候对于寻找分片大小很有用)。

试着发送大一些的ping报文，然后通过tcpdump查看IP序列号。

traceroute是一个工具，通过它可以获得关于到特定主机的路由信息。Traceroute发送IP报文，然后增加TTL值(事实上是每个TTL发送3个报文)，然后侦听对于每一跳的ICMP "TIME_EXCEDDED"响应，这些跳对应到达主机的路径。关于traceroute一些重要信息有:

* 每一跳描述一个接口(从那些发送time-exceded消息的主机那里获取),而不是主机或者路由器。一条路径可能经过超过一个路由器。
* 发送time-exceded消息的源头被设置为返回路径出接口的IP地址，而不是报文达到的接口，这会产生一些奇怪的结果。
* traceroute结果的失败也并不表明在那一点出现的失败(也许是必须的)。比如它可能标识着反向路径出现了失败。另外，"*"也不必定表示一个失败条件，比如这也许意味着，那个路由器不响应ICMP消息。

请注意，大多数机器上的 traceroute 版本不支持“-A”选项。您可以手动执行此操作（如上所示），查找 RPM 或适合您机器的现有软件包，或者从http://nms.lcs。mit.edu/6.8/other/traceroute-nanog.tar.gz 下载“Nanog Tracerout”，并根据您的特定机器进行编译（请注意，traceroute 要么需要超级用户权限才能运行，要么必须将其设置为“setuid root”，否则您将收到权限错误）


## 4. Perl

在本部分中，我们将看到如何使用 Perl（一种流行的脚本语言）对 tepdump 文件执行两个分析任务：
* 计算从一台主机到另一台主机传输的字节数
* 绘制ACK跟踪图


