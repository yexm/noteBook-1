---
title: http和https
tags: web,http,https
grammar_cjkRuby: true
grammar_flow: true
---


# Http和Https

## 概述及储备

C/S架构，应用层协议

### 端口划分

传输层协议：TCP，UDP，SCTP

IANA（地址规范化组织）：
- 0~1023，永久的分配给固定应用使用，特权端口；
- 1024~41951，不严格要求；
- 41952~60000， 客户端程序随机使用端口，动态/私有端口；其范围定义在/proc/sys/net/ipv4/ip_local_port_range中；

### BSD Socket

IPC的一种实现，相同/不同主机进程之间的通信机制。

Socket API封装了内核中套接字通信相关的系统调用，通过库调用方式提供给程序员。

**Socket API 分类**
- SOCK_STREAM: TPC 套接字
- SOCK_DGRAM: UDP套接字 
- SOCK_RAW: 裸套接字

**根据套接字地址格式划分：**
- AF_INET: Address Family,IPv4
- AF_INET6: IPv6
- AF_UNIX: 同一主机不同进程间基于套接字通信时使用的地址

![TCP通信机制](./images/TCP通信机制.jpg)

***TCP/IP有限状态机：***

FSM：

``` elixir
CLOSED->LISTEN
	  ->SYN_SENT
	  ->SYN_RECV
	  ->ESTABLISHED
	  ->FIN_WAIT1
	  ->CLOSE_WAIT
	  ->FIN_WAIT2
	  ->LASK_ACK
	  ->TIME_WAIT
	  			->CLOSED
```


