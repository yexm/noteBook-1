---
title: ROOK CEPH 故障节点恢复流程 
tags: rook,ceph,故障
grammar_cjkRuby: true
---


# 故障恢复流程

##### STEP 1
在dashboard或tools中删除故障osd，在dashboard中直接purge，其他方式可能无法移除节点，但使用这种方式可能会造成部分pg的unknown。因此创建pool的时候，故障域要设置为host，当一个host中有多个osd的时候，尤为重要。

##### STEP 2
将故障osd从kubernetes资源中移除，编辑cephclusters.ceph.rook.io资源（当自动部署ceph开启时，此操作无效）。

##### STEP 3
清理节点
```bash
sgdisk --zap-all /dev/sd?
ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %
rm -rf /dev/ceph-*
#删除过程中可能会遇到rbd映射的块设备仍然mount在系统上，需要处理并删除，/dev/mapper目录下的ceph-*文件同样需要检查，是否清理干净

rm -rf /var/lib/rook/osd*
#清理掉旧的osd信息
```
##### STEP 4
重新配置cephclusters.ceph.rook.io资源

##### STEP 5
重启rook-ceph-operator