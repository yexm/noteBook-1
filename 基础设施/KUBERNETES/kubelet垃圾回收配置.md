# Configuring kubelet Garbage Collection





## Imange Collection

Kubernetes通过kubelet集成的cadvisor进行镜像的回收，有两个参数可以设置：

* --image-gc-high-threshold
* --image-gc-low-threshold

当用于存储镜像的磁盘使用率达到百分之--image-gc-high-threshold时将触发镜像回收，删除最近最久未使用（LRU，Least Recently Used）的镜像直到磁盘使用率降为百分之--image-gc-low-threshold或无镜像可删为止。默认--image-gc-high-threshold为90，--image-gc-low-threshold为80。

## Container Collection

容器的回收有三个参数可设置：

* --minimum-container-ttl-duration
* --maximum-dead-containers-per-container
* --maximum-dead-containers

从容器停止运行时起经过--minimum-container-ttl-duration时间后，该容器标记为已过期将来可以被回收（只是标记，不是回收），默认值为1m0s。一般情况下每个pod最多可以保留--maximum-dead-containers-per-container个已停止运行的容器集，默认值为2。整个node节点可以保留--maximum-dead-containers个已停止运行的容器，默认值为100。

如果需要关闭容器的垃圾回收策略，可以将--minimum-container-ttl-duration设为0（表示无限制），--maximum-dead-containers-per-container和--maximum-dead-containers设为负数。

