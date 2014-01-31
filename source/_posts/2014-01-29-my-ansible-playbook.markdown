---
layout: post
title: "我的Ansible playbook"
date: 2014-01-29 22:08
comments: true
categories: DevOps
---

**前提**，安装好Ansible，参考我的上一篇博客，[Ansible 快速入门](http://www.yanjiuyanjiu.com/blog/20140127)

    ---
    - hosts: all
      sudo: True
      remote_user: work
      vars:
        ant_version: 1.9.3
        maven_version: 3.1.1
        scala_version: 2.10.3
        sbt_version: 0.12.4
    
      tasks:
      ########## for CentOS and RedHat ##########
        - name: Install the libselinux-python package  # because ansible needs it
          yum: name=libselinux-python state=installed
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
        - name: Disable SELinux in conf file
          selinux: state=disabled
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
        - name: update YUM repositories
          shell: 'yum -y update'
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
        - name: Install OpenJDK
          yum: name=java-1.7.0-openjdk-devel state=present
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
        - name: Set JAVA_HOME environment variable
          lineinfile: dest='/etc/profile' regexp='^#?\s*export JAVA_HOME=(.*)$' line='export JAVA_HOME=/usr/lib/jvm/java' state=present
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
        - name: install the 'Development tools' package group
          yum: name="@Development tools" state=present
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
        - name: upgrade all packages
          shell: 'yum -y upgrade'
          when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    
    
      ########## for Ubuntu and Debian ##########
        - name: Run "apt-get update" to update the source list
          apt: update_cache=yes
          when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
    
        - name: Install OpenJDK
          apt: pkg=openjdk-7-jdk state=present
          when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
    
        - name: Set JAVA_HOME environment variable
          lineinfile: dest='/etc/profile' regexp='^#?\s*export JAVA_HOME=(.*)$' line='export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64' state=present
          when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
    
        - name: Install build-essential
          apt: pkg=build-essential state=present
          when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
    
        - name: Update all packages to the latest version
          apt: upgrade=dist
          when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
    
    
      ########## Common opreations for all OS ##########
        # Create local directories
        - file: path=~/local/bin state=directory
          sudo: no
        - file: path=~/local/sbin state=directory
          sudo: no
        - file: path=~/local/src state=directory
          sudo: no
        - file: path=~/local/opt state=directory
          sudo: no
        - file: path=~/local/var state=directory
          sudo: no
        - lineinfile: dest=~/.bashrc regexp='^#?\s*export PATH=(.*)/local/bin(.*)$' line="export PATH=$PATH:$HOME/local/bin" state=present
          sudo: no
        - lineinfile: dest=~/.bashrc regexp='^#?\s*export PATH=(.*)/local/sbin(.*)$' line="export PATH=$PATH:$HOME/local/sbin" state=present
          sudo: no
    
        - name: Set CLASSPATH and PATH environment variables
          lineinfile: $item
          with_items:
            - dest='/etc/profile' regexp='^#?\s*export CLASSPATH=(.*)$' line='export CLASSPATH=.:$JAVA_HOME/lib/*.jar:$JAVA_HOME/jre/lib/*.jar' state=present
            - dest='/etc/profile' regexp='^#?\s*export PATH=(.*)JAVA_HOME(.*)$' line="export PATH=$PATH:$JAVA_HOME/bin" state=present
    
        ###### Since Ant in yum and apt is too old, download the .tar.bz2 file and install it
        # Install Apache Ant
        - name: Download Apache Ant
          get_url: url=http://mirror.cc.columbia.edu/pub/software/apache//ant/binaries/apache-ant-{{ant_version}}-bin.tar.bz2 dest=/tmp/apache-ant-{{ant_version}}-bin.tar.bz2
        - name: Untar Ant
          shell: chdir=/tmp creates=/opt/apache-ant-{{ant_version}} tar -jxf apache-ant-{{ant_version}}-bin.tar.bz2 -C /opt
        - lineinfile: dest=/etc/profile regexp='^#?\s*export ANT_HOME=(.*)$' line='export ANT_HOME=/opt/apache-ant-{{ant_version}}' state=present
        - lineinfile: dest=/etc/profile regexp='^#?\s*export PATH=(.*)ANT_HOME(.*)$' line="export PATH=$PATH:$ANT_HOME/bin" state=present
    
        # Install Apache Maven, since there is no Maven package in yum and apt repo
        - name: Download Apache Maven
          get_url: url=http://apache.claz.org/maven/maven-3/3.1.1/binaries/apache-maven-{{maven_version}}-bin.tar.gz dest=/tmp/apache-maven-{{maven_version}}-bin.tar.gz
        - name: Untar Maven
          shell: chdir=/tmp creates=/opt/apache-maven-{{maven_version}} tar -zxf apache-maven-{{maven_version}}-bin.tar.gz -C /opt
        - lineinfile: dest=/etc/profile regexp='^#?\s*export MAVEN_HOME=(.*)$' line='export MAVEN_HOME=/opt/apache-maven-{{maven_version}}' state=present
        - lineinfile: dest=/etc/profile regexp='^#?\s*export PATH=(.*)MAVEN_HOME(.*)$' line="export PATH=$PATH:$MAVEN_HOME/bin" state=present
    
        # Install the scala compiler, because the scala compiler in yum and apt repo is too old
        - name: Download Scala
          get_url: url=http://www.scala-lang.org/files/archive/scala-{{scala_version}}.tgz dest=/tmp/scala-{{scala_version}}.tgz
        - name: Untar Scala
          shell: chdir=/tmp creates=/opt/scala-{{scala_version}} tar -zxf scala-{{scala_version}}.tgz -C /opt
        - lineinfile: dest=/etc/profile regexp='^#?\s*export SCALA_HOME=(.*)$' line='export SCALA_HOME=/opt/scala-{{scala_version}}' state=present
        - lineinfile: dest=/etc/profile regexp='^#?\s*export PATH=(.*)SCALA_HOME(.*)$' line="export PATH=$PATH:$SCALA_HOME/bin" state=present
    
        # Install sbt
        - name: Download sbt
          get_url: url=http://repo.scala-sbt.org/scalasbt/sbt-native-packages/org/scala-sbt/sbt//{{sbt_version}}/sbt.tgz dest=/tmp/sbt-{{sbt_version}}.tgz
        - name: Untar sbt
          shell: chdir=/tmp creates=/opt/sbt-{{sbt_version}} tar -zxf sbt-{{sbt_version}}.tgz -C /opt
        - name: Rename sbt directory
          shell: chdir=/opt creates=/opt/sbt-{{sbt_version}} mv sbt/ sbt-{{sbt_version}}/
        - lineinfile: dest=/etc/profile regexp='^#?\s*export SBT_HOME=(.*)$' line='export SBT_HOME=/opt/sbt-{{sbt_version}}' state=present
        - lineinfile: dest=/etc/profile regexp='^#?\s*export PATH=(.*)SBT_HOME(.*)$' line="export PATH=$PATH:$SBT_HOME/bin" state=present

