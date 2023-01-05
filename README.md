---
description: ansible自动化运维工具
---

# Ansible部署配置

## ansible官网：

```bash
https://www.ansible.com/
```

## 1、什么是ansible

```
Ansible 是一个 IT 自动化的“配置管理”工具，自动化主要体现在 Ansible 集成了丰富模块，以及强大的功能组件，可以通过一个命令行完成一系列的操作。进而能减少我们重复性的工作，以提高工作的效率。
```

## 2、ansible的安装

```bash
#配置yum源
echo '[ansible]
name=ansible
baseurl=https://mirror.tuna.tsinghua.edu.cn/epel/7/x86_64/
gpgcheck=0' > /etc/yum.repos.d/ansible.repo

#清理旧缓存重新生成
yum clean all && yum makecache

#开始安装
yum install ansible -y
```

## 3、配置文件解读

```
/etc/ansible/ansible.cfg ：主配置文件，配置 ansible 工作特性
/etc/ansible/hosts ：配置主机清单文件
/etc/ansible/roles/ ：存放 ansible 角色的目录
```

```
[root@manager ~]# cat /etc/ansible/ansible.cfg
[defaults] 
#inventory = /etc/ansible/hosts #主机列表配置文件 
#library = /usr/share/my_modules/ #库文件存放目录 
#remote_tmp = ~/.ansible/tmp #临时py文件存放在远程主机目录
#local_tmp = ~/.ansible/tmp #本机的临时执行目录

#forks = 5 #默认并发数 
#sudo_user = root #默认sudo用户 
#ask_sudo_pass = True #每次执行是否询问sudo的ssh密码
#ask_pass = True #每次执行是否询问ssh密码 
#remote_port = 22 #远程主机端口 
#host_key_checking = False #检查对应服务器的host_key， 建议取消 
#log_path = /var/log/ansible.log #ansible日志，建议启用 
[privilege_escalation] #如果是普通用户则需要配置提权 
#become=True 
#become_method=sudo 
#become_user=root 
#become_ask_pass=False
```

<pre class="language-bash"><code class="lang-bash">hosts主机清单文件：
[webservers]
10.0.0.15 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'
10.0.0.16 ansible_ssh_port=22 ansible_ssh_port=root ansible_ssh_pass='123456'
<strong>#也可提前配置密钥免密认证；配置完后可以省略ansible_ssh_port、ansible_ssh_port、ansible_ssh_pass#··
</strong></code></pre>

补充：

```bash
[root@manager ~]# ssh-keygen -t rsa    #生成密钥
[root@manager ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.0.0.1    #发送至目标主机
```

## 4、ansible常用命令

```bash
命令格式：
ansible <host-pattern> [-m module_name] [-a args]

[root@centos7-hml ~]# ansible webservers -m shell -a "df -h"
###ansible对webservers分组下的设备运行shell模块，执行df -h的命令
```

```bash
# 指定操作所有的组 
ansible all -m ping

# 通配符
ansible "*" -m ping
ansible 10.0.0.* -m ping

# 与(:&)：在webservers组；并且在dbservers中的主机；
ansible "webservers:&dbservers" -m ping

# 或(:)：在webservers组，或者在appservers中的主机;
ansible "webservers:appservers" -m ping 

# 非(:!)：在webservers组，但不在apps组中的主机 
ansible 'webservers:!apps' -m ping 

# 正则表达式
ansible "~(web|db).*\.oldxu\.com" -m ping
```

## 5、ansible模块

```bash
ping、yum、template、copy、user、group、service、raw、command、shell、script

模块繁多，ansible常用模块有raw、command、shell
三种模块的区别作了解即可
shell模块调用的/bin/sh指令执行
command模块不是调用的shell的指令，所以没有bash的环境变量
raw很多地方和shell类似，更多的地方建议使用shell和command模块。但是如果是使用老版本python，需要用到raw，又或者是客户端是路由器，因为没有安装python模块，那就需要使用raw模块了
```

## **6、ansible Playbook**

```bash
playbook 是一个 由 yaml 语法编写的文本文件，它由 play 和 task 两部分组成。
play ： 主要定义要操作主机或者主机组
task ：主要定义对主机或主机组具体执行的任务，可以是一个任务，也可以是多个任务（模块）
```

```bash
#1、定义play
#2、定义task

- hosts: webservers
  tasks:
    - name: Install Httpd Server		# - name 定义
      yum:
        name: httpd
        state: present

    - name: Configure Httpd Server
      copy:
        src: /etc/httpd/conf/httpd.conf
        dest: /etc/httpd/conf/httpd.conf
        owner: "root"
        group: "root"
        mode: "0644"
        backup: yes
      notify: Restart Httpd Server		#notify: 定义一个名为Restart Httpd Server的触发器，只										 要copy发生改变，就会调用Restart Httpd Server名字的模块

    - name: Systemd Httpd Server
      systemd:
        name: httpd
        state: started
        enabled: yes

  handlers:								#handlers:被调用的位置
    - name: Restart Httpd Server
      systemd:
        name: httpd
        state: restarted
```

7、本文由huiml编写，觉得不错请打赏下！

![](.gitbook/assets/lQDPJxXdycyehczNBGTNAzywYY0eIG3hlq8DsfPMvEBZAA\_828\_1124.jpg)
