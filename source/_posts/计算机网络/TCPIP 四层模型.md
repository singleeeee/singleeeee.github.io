---
title: TCP/IP 四层模型
categories: 计算机网络
---

# TCP/IP 四层模型

## 应用层

- 负责应用与应用之间的通信
- HTTP、FTP、Telnet、DNS、SMTP等
- 应用层是不用去关心数据是如何传输的

## 传输层

> 应用层的数据包会传给传输层，**传输层**（*Transport Layer*）是为应用层提供网络支持的。

- 在传输层会有两个传输协议，分别是 `TCP` 和 `UDP`。
- TCP 的全称叫传输控制协议（*Transmission Control Protocol*）。TCP 相比 UDP 多了很多特性，流量控制、超时重传、拥塞控制等，这些都是为了保证数据包能`可靠`地传输给对方。
- 当传输层的数据包大小超过 MSS（TCP 最大报文段长度） ，就要将数据包分块, 这样即使中途有一个分块丢失或损坏了，只需要重新发送这一个分块，而不用重新发送整个数据包。
- 一台设备上可能会有很多应用在接收或者传输数据，因此需要用一个编号将应用区分开来，这个编号就是**端口**。

## 网络层

- 网络层最常使用的是` IP 协议`（*Internet Protocol*），IP 协议会将传输层的报文作为`数据部分`，再加上 `IP 包头`组装成 IP 报文
- 如果 IP 报文大小超过 MTU（以太网中一般为 1500 字节）就会**再次进行分片**，得到一个即将发送到网络的 IP 报文。
- 我们一般用 IP 地址给设备进行编号，对于 IPv4 协议， IP 地址共 32 位

## 网络接口层



# OSI七层模型

## 应用层

## 表示层

## 会话层

## 传输层

## 网络层

## 数据链路层

## 物理层







