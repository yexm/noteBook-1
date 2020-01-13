---
title: Selinux
tags: linux,security,selinux
grammar_cjkRuby: true
---

Secure Enhanced Linux(MAC强制访问控制机制的一种实现)

#### 两种工作级别

- strict：每个进程都受到selinux的控制
- targeted：仅有限个进程收到selinux控制，只监控容易收到入侵的进程


#### 工作机制

机制概述（三元素）：
subject	operation object

- subject：进程
- object：进程、文件
- operation：【文件：open,read,write,close,chown,chmod】

同一区域下的主体，访问客体的访问控制标签。

SElinux为每个文件及进程提供了安全标签。

由五段组成，但后两段没有意义，只看前三段

``` bash
user:role:tyep 		
#user：为selinux的用户
#role：角色
#type：类型(域)，关键
```
**SElinux规则库：**
规则：哪种域能访问哪种或哪些文件/进程的类型。被规则库拒绝后，将记录日志。***规则库没有明确指定均拒绝。***


**SElinux Bool功能设置：**

```bash
getsebool	#查看现有bool功能开关状态；
setsebool	#设置bool功能开关；
getenforce	#获取SElinux当前状态；
setenforce 0|1	#0设置为permissive， 1设置为enforcing，重启无效；

chcon	#重新编辑文件安全上下文标签
       chcon [OPTION]... CONTEXT FILE...
       chcon [OPTION]... [-u USER] [-r ROLE] [-l RANGE] [-t TYPE] FILE...
       chcon [OPTION]... --reference=RFILE FILE...
	   
resotrecon #还原文件安全上下文标签

```

参考《SElinux权威指南》