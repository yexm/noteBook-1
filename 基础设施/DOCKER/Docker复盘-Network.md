# Docker Replay-Network

先感慨一句，发展更新太快了，我上次培训的时候有好多东西都没有呢，这段时间忙于生产环境容器化，闲下来重新复盘一下doc。开整！！！

## 1  Overview

### Network drivers

* bridge，docker0与自定义桥接网络，用于独立container与外部通信
* host， 使用宿主网络
* overlay， Swarm，容器跨主机通信原生解决方案
* macvlan，分配MAC地址
* none， 禁用网络
* Network Plugins， flanel，calico等

## 2  Use bridge networks

### 2.1 用户定义bridge和default bridge的区别

* 用户定义bridge提供了更好的容器间的隔离与互通控制；
* 用户定义bridge提供了容器间的自动DNS
* Container可以动态加入或移出用户定义bridge
* 每个用户定义bridge都创建了一个可配置的bridge

关于官方文档中Container之间link的需求，现在看来已经是多余的了。

* _**default bridge不能通过Dockername来解析连接其他container**_

### 2.2  管理user-defined bridge

```
docker network create
【可用选项】
--ip-masq
--icc
--ip
--mtu
--gateway
--ip-range
--internal
--ipv6
--subnet
```

### 2.3  连接/移除user-defined bridge

```
docker network connect my-net my-nginx

docker network disconnect my-net my-nginx
```

## 3  Overlay 网络

构建docker集群间container通信网络。

当初始化swarm集群时，自动创建两个网络：

* ingress，处理控制及数据传输；
* docker\_gwbridge，Docker Deamon之间的通信桥梁。

可以使用--attachable标记，创建ingress网络，使得独立的container可用ingress网络

## 4  Macvlan networks

适用于需要连接物理网络的应用。可能会损害物理网络

### 创建macvlan network

#### Bridge mode

```
$ docker network create -d macvlan  \
  --subnet=192.168.32.0/24  \
  --ip-range=192.168.32.128/25 \
  --gateway=192.168.32.254  \
  --aux-address="my-router=192.168.32.129" \
  -o parent=eth0 macnet32
```

#### 802.1q trunk bridge mode

```
$ docker network  create  -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50
```

#### Use an ipvlan instead of macvlan

```
$ docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254  \
    --gateway=192.168.212.254  \
     -o ipvlan_mode=l2 ipvlan210
```

刚刚接触docker时，没有对于macvlan的支持，没有尝试过，官网也建议可用覆盖或桥接网络时，不要使用MacVlan

