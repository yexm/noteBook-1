Linux NameSpace
意义：内核级别的环境隔离方法，虚拟化，容器化技术的基础

简介：实际上是对进程的一种封装，是其在namespaces内误认为自己拥有所有全局资源

|分类|系统调用参数|内核版本|
|---|---|---|
|Mount namespaces|CLONE_NEWNS|Linux2.4.19|
|UTS namespaces| CLONE_NEWUTS|Linux2.6.19|
|IPC namespaces|CLONE_NEWIPC|Linux2.6.19|
|PID namespaces|CLONE_NEWPID|Linux2.6.24|
|Network namespaces|CLONE_NEWNET|Linxu2.6.24~2.6.29|
|User namespaces|CLONE_NEWUSER|Linux2.6.23~3.8|

**UTS:**主要目的是独立出主机名和网络信息服务，系统标识符，“nodename”和“domain name”

**IPC:**进程间通信，共享内存、信号量、消息队列等方法

**PID:**容器内外的进程PID可能相同，需要隔离

**Mount:**容器与系统挂载点分离，使用umount()卸载掉系统的全局挂载点，为进程仅挂在namespace相关的挂载点。

**Network:**不同namespces中可以有自己的网络设备

**USER:**隔离用户和组ID