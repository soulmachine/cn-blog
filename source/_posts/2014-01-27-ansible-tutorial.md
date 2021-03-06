---
layout: post
title: "Ansible 快速入门"
date: 2014-01-27 12:10
comments: true
categories: DevOps
---

Ansible 是一个比Puppet, Chef 更轻量的provisioning 工具，不需要启动daemon进程。这点跟跟pssh差不多，但是比pssh更加强大。

##前提
* 所使用的remote_user要能够SSH无密码登录到所有机器，配置方法见[SSH无密码登录的配置](http://www.yanjiuyanjiu.com/blog/20120102/)
* remote_use sudo的时候不需要密码，配置方法如下，

        sudo chmod +w /etc/sudoers
        sudo vim /etc/sudoers

    找到 `root    ALL=(ALL:ALL) ALL`，在下面添加一行

       username    ALL=(ALL:ALL) NOPASSWD:ALL

    保存退出，然后恢复为只读，

        sudo chmod +w /etc/sudoers 

如果忘记了以上两点，运行任何ansible命令的时候，会卡住不动很久。

如果发现在 "GATHERING FACTS"这里卡住，多半是sudo需要密码，试试加上-K选项，例如`ansible-playbook -K playbook.yml`，参考[Running ansible on local Linux desktop hangs on Gathering Facts](https://groups.google.com/forum/#!topic/ansible-project/FL0mxyxOo4M)。

`-vvvv`表示调试模式，加上后会输出很多中间信息，帮助你调试。

##1. 安装Ansible
只需要在一台机器上安装，其他机器不需要安装任何东西，这就是ansible比puppet, chef方便的地方。

    sudo yum install ansible

或者

    sudo apt-get install ansible

在`/etc/ansible/hosts`添加想要操作的机器(这个`hosts`文件也叫做[Inventory](http://docs.ansible.com/intro_inventory.html))，且这些机器都是能[SSH无密码登录的](http://www.yanjiuyanjiu.com/blog/20120102)，然后测试一下：

    ansible all -a "/bin/echo hello"

如果都成功了，说明安装成功。

使用ansible有两种方式：Ad-hoc command 和 playbook。前者用于临时类批量操作，后者用于配置管理，类似与Puppet。

##2. Ad-Hoc Commands
Ad-hoc命令的形式一般如下：

    ansible groupname -m module -a arguments

<!-- more -->
例如：

    ansible all -m yum -a "name=wget state=present"

参考[Introduction To Ad-Hoc Commands](http://docs.ansible.com/intro_adhoc.html)

##3. Playbook

对于稳定的配置，就要使用playbook了。

一个playbook由多个play组成，一个play由多个task组成，参考[Intro to Playbooks](http://docs.ansible.com/playbooks_intro.html)。

一个playbook的文件内容通常是如下形式：

    ---
    - hosts: groupname
      remote_user: yourname
      sudo: yes
    
      tasks:
        - task1
        - task2

一个play的文件内容通常是如下形式：

    - task1
    - task2
        
例如，批量安装wget和gcc的playbook，可以这么写：

    ---
    - hosts: all
      remote_user: username
      sudo: yes
        
      tasks:
        - yum: name=wget state=present
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
        - yum: name=gcc state=present
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'

这个play包含了2个task。执行一下这个playbook，看看效果:

    ansible-playbook wget_playbook.yml

接下来再举一个例子，我们写两个play，`play_debian.yml`和`play_centos.yml`，以及一个playbook，`playbook.yml`。

play_cenos.yml:

    - yum: name=wget state=present
    - yum: name=gcc state=present

play_deban.yml:

    - apt: pkg=wget state=present
    - apt: pkg=gcc state=present

playbook.yml:

    ---
    - hosts: all
      remote_user: username
      sudo: yes
    
      tasks:
        - include: play_centos.yml
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
        - include: play_debian.yml
          when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'


执行这个playbook:

    ansible-playbook playbook.yml

##4. 进阶
想要进一步了解ansible，可以学习官网的例子, [ansible examples](https://github.com/ansible/ansible-examples/)。

一定要仔细阅读官网给出的最佳实践规范，[Best Practices](http://docs.ansible.com/playbooks_best_practices.html)。
