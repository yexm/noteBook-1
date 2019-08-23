# CentOS 6.X PHP 5.3 升级

CentOS PHP

## 1.确认当前PHP版本

```bash
rpm -qa |grep php
```

## 2.安装激活REMI和EPEL源

```bash
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm && rpm -Uvh epel-release-latest-6.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm && rpm -Uvh remi-release-6*.rpm
# 编辑REMI源，enabled=1
```

## 3.更新PHP

```bash
yum -y update php*
# 测试验证更新后的版本
php -v
```