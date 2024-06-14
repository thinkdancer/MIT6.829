# Networking Tools

## Overview
在这个指导手册中，我们会覆盖网络研究中经常使用的工具（更不用说在现实世界中调试使用了）。我们从netstat和tcpdump开始，通过查看一个活跃的TCP连接，并熟悉它们的输出结果，以此来简要介绍如何使用它们中的每一个。我们将会使用netstat来描述TCP状态机的每一个元素。

接下来，我们会学习一组基于查找的工具，dig和whois,它们提供了主机的DNS信息，并分别提供关于名字和网络的注册信息。

然后，我们会转换到标准网络测量工具，ping以及更加有趣的traceroute,并描述它们如何工作，以及它们能（不能）告诉你什么。

最后，对于你们中不知道perl的人，我将展示如何“将事情黏连起来”。具体方法是展示一个简单的perl脚本，他读取我们的trace结果，并计算到目的IP地址和AS网络号的字节数。

## 1. tcpdump and netstat

## 2. Lookup Tools: dig and whois

## 3. Measurement Tools : ping and traceroute

## 4. Perl



