## CPU选型

* 元数据服务器，4核+
* OSD服务器，2核+
* Monitor服务器， CPU不敏感

## RAM内存

永远都是越多越好，数据恢复期间，每1TB数据每进程，需要1GB的内存。

## 数据存储

btrfs尚不具备生产环境使用的稳定性，但其可进行日志、数据同步读写。

建议使用xfs，次之ext4，但是不具备日志、数据同步读写



Ceph 可以运行在廉价的普通硬件上，小型生产集群和开发集群可以在一般的硬件上。

| 进程 | 条件 | 最低建议 |
| :--- | :--- | :--- |
| ceph-osd | Processor | 1x 64-bit AMD-641x 32-bit ARM dual-core or better1x i386 dual-core |
|  | RAM | ~1GB for 1TB of storage per daemon |
|  | Volume Storage | 1x storage drive per daemon |
|  | Journal | 1x SSD partition per daemon \(optional\) |
|  | Network | 2x 1GB Ethernet NICs |
| ceph-mon | Processor | 1x 64-bit AMD-64/i3861x 32-bit ARM dual-core or better1x i386 dual-core |
|  | RAM | 1 GB per daemon |
|  | Disk Space | 10 GB per daemon |
|  | Network | 2x 1GB Ethernet NICs |
| ceph-mds | Processor | 1x 64-bit AMD-64 quad-core1x 32-bit ARM quad-core1x i386 quad-core |
|  | RAM | 1 GB minimum per daemon |
|  | Disk Space | 1 MB per daemon |
|  | Network | 2x 1GB Ethernet NICs |

生产级

一个最新（ 2012 ）的 Ceph 集群项目使用了 2 个相当强悍的 OSD 硬件配置，和稍逊的监视器配置。

| Configuration | Criteria | Minimum Recommended |
| :--- | :--- | :--- |
| Dell PE R510 | Processor | 2x 64-bit quad-core Xeon CPUs |
|  | RAM | 16 GB |
|  | Volume Storage | 8x 2TB drives. 1 OS, 7 Storage |
|  | Client Network | 2x 1GB Ethernet NICs |
|  | OSD Network | 2x 1GB Ethernet NICs |
|  | Mgmt. Network | 2x 1GB Ethernet NICs |
| Dell PE R515 | Processor | 1x hex-core Opteron CPU |
|  | RAM | 16 GB |
|  | Volume Storage | 12x 3TB drives. Storage |
|  | OS Storage | 1x 500GB drive. Operating System. |
|  | Client Network | 2x 1GB Ethernet NICs |
|  | OSD Network | 2x 1GB Ethernet NICs |
|  | Mgmt. Network | 2x 1GB Ethernet NICs |



