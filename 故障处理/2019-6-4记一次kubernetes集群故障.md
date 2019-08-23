---
title: 2019-6-4记一次kubernetes集群故障
tags: 故障,kubernetes,网络通信
grammar_cjkRuby: true
---

### 现象：
集群内POD无法解析外部域名
初期判断失误，误以为node节点网络问题
后发现POD无法解析域名，但是可以ping通相应的IP地址， 判断为DNS问题
外部DNS可以ping通，但内部的10.96.0.10的DNS Service地址无法正常访问
前期故障排查过程中发现存在部分master节点与故障node节点POD无法通信问题
基于以上过程判断，master与故障node节点的路由表出现异常

### 解决方案：
重启master节点， 恢复master 与故障节点flannel的路由， 再次尝试从故障节点pod内访问kuberDNS，可以正常访问，至此，问题解决。


### 说明：
故障排查过程中，一直以为是flannel网络的问题，对于此问题的解决，快速方法重新加入集群。
但这一操作需要对节点中的cni0、flannel1.1、docker0进行手动删除，然后重启docker及kubelet