---
title: ansible
tags: 运维,自动化,ansible
renderNumberedHeading: true
grammar_cjkRuby: true
---

# Inventory

ansible资产管理，默认文件位置`/etc/ansible/hosts`

## 主机组

主机组最简单的表达形式

``` vim
[GROUPNAME]
asset01.liuzhi.com
10.10.100.129

# 非默认端口号
10.10.100.130:1522

# 资产别名表达方式
aliasAsset ansible_ssh_port=22 ansible_ssh_host=10.10.100.131

# 自动序号资产表示方法
worker[01:50].liuzhi.com
node-[a:f].liuzhi.com

# 指定连接类型和用户名
[targets01]
localhost	ansible_connection=local
other1.liuzhi.com 	ansible_connection=ssh	ansible_ssh_user=user01
other2.liuzhi.com	ansible_connection=ssh	ansible_ssh_user=user02
```


## 主机变量

可以为指定主机设置变量，在ansible-playbooks中可以调用。

主机变量的定义方式如下：

``` vim

[VARSGROUP]
# http_port和maxClient可以在playbooks中调用
host1.liuzhi.com	http_port=80	maxClient=1000
host2.liuzhi.com	http_port=8080	maxClient=2000

```

组变量的定义方式如下：

``` vim

[group01]
host01
host02

[group01:vars]
ntp_servers=ntp.liuzhi.com
proxy=proxy01.liuzhi.com:142542

```

组和组合定义方式如下：

``` vim
[group1]
host1
host2

[group2]
host3
host4

# group1和group2是groupf的子组，将继承groupf定义的vars
[groupf:children]
group1
group2

[groupf:vars]
replicas=2
```
## Inventory最佳实践
使用分割文件未host配置变量是一种良好的组织配置文件的习惯。
***应用实践？？？***


# Playbook

## Playbook基础

### 主机与用户

``` YAML
---
- hosts: webservers, dbs
  remote_user: root
```
再者,在每一个 task 中,可以定义自己的远程用户:

``` YAML
---
- hosts: webservers
  remote_user: root
  tasks:
    - name: test connection
      ping:
      remote_user: yourname
```

### TASK列表
每一个 play 包含了一个 task 列表（任务列表），task支持幂等，重复多次执行 playbook 也很安全.

**Handlers: 在发生改变时执行的操作**

实例，当文件变动出发两个服务的重启：

``` YAML
- name: template configuration file
  template: src=template.j2 dest=/etc/foo.conf
  notify:
     - restart memcached
     - restart apache

handlers:
    - name: restart memcached
      service:  name=memcached state=restarted
    - name: restart apache
      service: name=apache state=restarted
```

## Playbook Roles和Include语句
为了让文件是可以方便去重用的，所以需要重新去组织这些文件。基本上，使用 include 语句引用 task 文件的方法，可允许你将一个配置策略分解到更小的文件中。

### Include和鼓励重用
重用，通过多个playbook Include同一个task列表文件来实现。

实例如下：

``` YAML
---
# possibly saved as tasks/foo.yml

- name: placeholder foo
  command: /bin/foo

- name: placeholder bar
  command: /bin/bar
```

Include 指令看起来像下面这样，在一个 playbook 中，Include 指令可以跟普通的 task 混合在一起使用:
  

``` YAML
  tasks:

  - include: tasks/foo.yml
```

你也可以给 include 传递变量。我们称之为 **‘参数化的 include’**。

``` YAML
tasks:
  - include: wordpress.yml wp_user=timmy
  - include: wordpress.yml wp_user=alice
  - include: wordpress.yml wp_user=bob
```
handlers模块的使用方式与Task类似。


### Role
Role是一种ansible playbook的文件组织方式
一般组织结构为：

``` bash
roles
		|- rolename
		   |- files/
		   |- templates/
		   |- tasks/
		   |- handlers/
		   |- vars/
		   |- defaults/
		   |- meta/
```

playbook文件配置如下：

```YAML
---
- hosts: webservers
  roles:
     - common
     - webservers
```

可以使用参数化的role

```YAML
---

- hosts: webservers
  roles:
    - common
    - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
    - { role: foo_app_instance, dir: '/opt/b',  port: 5001 }
```

可使用条件出发指定role

``` YAML 
---

- hosts: webservers
  roles:
    - { role: some_role, when: "ansible_os_family == 'RedHat'" }
```

可以为role设定tag

``` YAML
---

- hosts: webservers
  roles:
    - { role: foo, tags: ["bar", "baz"] }
```

使用roles的playbook仍然可以使用tasks，但tasks将在所有roles执行完毕之后执行，如果需要调整顺序，可以通过使用，pre_tasks和post_tasks实现循序的控制。

``` YAML
---

- hosts: webservers

  pre_tasks:
    - shell: echo 'hello'

  roles:
    - { role: some_role }

  tasks:
    - shell: echo 'still busy'

  post_tasks:
    - shell: echo 'goodbye'
```

**Role默认变量**

在`defaults/mail.yml`中设置的变量拥有全局最低的优先级，可被其他地方的变量所覆盖。

**Role依赖**

在`meta/mail.yml`可以引入其他依赖的Role，这将自动的拉取依赖的Role到现在使用的Role中。
实例：

```YAML
---
dependencies:
  - { role: common, some_parameter: 3 }
  - { role: apache, port: 80 }
  - { role: postgres, dbname: blarg, other_parameter: 12 }
```

***角色依赖可以通过绝对路径直接引用。***

角色可以通过源码仓库和tar包来指定，如下所示：

``` YAML
---
dependencies:
  - { role: 'git+http://git.example.com/repos/role-foo,v1.1,foo' }
  - { role: '/path/to/tar/file.tgz,,friendly-name' }
```

依赖会被递归调用，如果依赖重复引用了某个Role该Role只会被执行一次。但这种默认行为可以被修改，通过设置`allow_duplicates: yes`来实现。

# Variables
请使用Python相同的变量定义方式。

## Inventory文件中定义变量
略
## 在playbook中定义变量

``` YAML
- hosts: webservers
  vars:
    http_port: 80
```
## 在Role中定义变量
定义在role组织的文件中

## 变量的使用
jinja2，python的变量引用方式`{{ 变量名 }}`， 可以使用jinja2的过滤属性，但是并不常用。

## Yaml陷阱
为了解决yaml文件解析过程中对{}理解为字典的问题，当所在行以引用变量开头时，应该使用`""`号将内容包含在内。

## Fact
可以直接应用Fact中包含的变量，如果不需要使用，可以设置关闭Fact，如果所示：

``` YAML
- hosts: whatever
  gather_facts: no
```

Fact缓存提供两种方式，redis和jsonfile，对于*大规模基础设施*的管理有必要进行相关配置。


## 注册变量

变量的另一个作用是在运行时保存命令的执行结果，以备后续调用。如下所示：

``` YAML
- hosts: web_servers

  tasks:

     - shell: /usr/bin/foo
       register: foo_result
       ignore_errors: True

     - shell: /usr/bin/bar
       when: foo_result.rc == 5
```
注册变量与facts的生命周期类似

## 运行时转递变量

Example：

``` YAML
---

- hosts: '{{ hosts }}'
  remote_user: '{{ user }}'

  tasks:
     - ...
```
运行时传递变量

```bash
ansible-playbook release.yml --extra-vars "hosts=vipers user=starbuck"
```

可通过`--extra-vars @a.json`，运行时应用json文件传递变量，也可以直接提供josn String

# 条件选择

## When语句

实例：

``` YAML
tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
```

==注意jinja2的过滤器的使用，会提供丰富的功能==

