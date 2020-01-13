# Linux 核心笔记

## Linux Basics

### 基本命令

#### 【命令】cd

```bash
cd -  #用以在当前目录与上次工作目录之间切换
```

#### 【快捷键】

```bash
ctrl+a  #光标定位至行首
ctrl+e  #光标定位至行尾
ctrl+l  #清屏
ctrl+u  #清除光标至行首的字符
ctrl+k  #清除光标至行尾的字符
```

#### 【命令】tee

```bash
COMMAND | tee /some/file     #命令输出定向到指定文件
```

#### 【命令】find

```bash

find [OPTIONS]  [查找起始路径]  [查找条件]  [处理动作]

    -type f/d/l/c/b/p/s
    -user
    -group
    -nouser
    -nogroup
    -name/iname/regex
    -size
    -atime/ctime/mtime
    -perm -mode
    [action]
    -ls
    -delete
    -exec COMMAND {} \;
    -ok COMMAND {} \;
```

### shell基础

#### 短路算法

```bash
COMMAND1&&COMMAND2      #当COMMAND1为假时，COMMAND2将不会执行，COMMAND1为真时，COMMAND2必须执行
COMMAND1||COMMAND2      #当COMMAND1为真时，COMMAND2不会执行，COMMAND1为假时，COMMAND2必须执行
```

## Linux 系统管理

### 设备管理

ext系列文件系统

```bash
dumpe2fs BLOCKDEVICE
```

xfs文件系统

```bash
xfs_info BLOCKDEVICE
```

### 文件系统

#### 【命令】mount的使用技巧

```bash
mount --bind srcDir destDir
```

### bash脚本编程

#### bool表达式

两种形式：

``` bash
[ expr ]  #表达式前后必须有空格

[[ expr ]]   #表达式前后必须有空格
```

三种类型：

- 数值表达式
- 字符表达式
- 文件表达式

##### 数值表达式

``` bash
-eq  #等于； [ $num1 -eq $num2 ]
-ne  #不等于；
-gt  #大于；
-ge  #大于等于；
-lt  #小于；
-le  #小于等于；
```

##### 字符表达式

``` bash
==  #等于；
!=  #不等于；
=~  #左侧字符串能否被右侧匹配；
>   #大于等于；
<   #小于；
-z "STRING"  #是否为空；
-n "STRING"  #是否不为空；

要用[[ ]],字符串要加引用。
```

##### 文件表达式

``` bash
文件存在
    -a  
    -e
存在及类型
    -b FILE  #是否存在，是否为块设备
    -c FILE  #是否存在，是否为字符设备
    -d FILE  #是否存在，是否为目录
    -f FILE  #是否存在，是否为普通文件
    -f FILE  #是否存在，是否为普通文件
    -h/L FILE  #是否存在，是否为符号连接
    -p FILE  #是否存在，是否为命名管道文件
    -S FILE  #是否存在，是否为Socket文件
文件权限
    -r
    -w
    -x
特殊权限测试
    -u  FILE  #是否存在并且 拥有suid权限
    -g  FILE  #是否存在并且 拥有sgid权限
    -k  FILE  #是否存在并且 拥有sticky权限

组合条件
方式一：
[ expr1 ] && [ expr2 ]  #&&，||，!
方式二：
[ expr1 -a expr2 ]      #-a, -o, !
```

#### 变量

##### 特殊变量

```bash
$0      #脚本文件路径本身
$#      #脚本参数个数
$*      #所有参数
$@      #所有参数
```

#### 流程控制

***实例：***通过参数传递一个用户名给脚本，此用户不存在则添加。

```bash
#!/bin/bash
#

if [ $# -lt 1 ]
then
    echo "At least one username."
    exit 2
fi

if grep "^$1\>" /etc/passwd &> /dev/null        #注意'\>'词尾锚定符
then
    echo "User $1 has exists."
else
    useradd $1
    echo $1 | passwd --stdin $1 &> /dev/null
    echo "Add user $1 finished."
fi
```