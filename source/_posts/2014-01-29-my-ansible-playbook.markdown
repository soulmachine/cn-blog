---
layout: post
title: "我的Ansible playbook"
date: 2014-01-29 22:08
comments: true
categories: 
---

	---
	- hosts: all
	  sudo: True
	  remote_user: work     # 要根据自己的集群更改
	
	  tasks:
	  ########## for CentOS and RedHat ##########
	    - name: Install the libselinux-python package
	      yum: name=libselinux-python state=installed
	      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'

	    # update YUM repositories
	    - shell: 'yum -y update'
	      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
	
	    - name: Install OpenJDK
	      yum: name=java-1.7.0-openjdk state=present
	      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
	
	    - name: install the 'Development tools' package group
	      yum: name="@Development tools" state=present
	      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
	
	    # upgrade all packages
	    - shell: 'yum -y upgrade'
	      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
	
	
	  ########## for Ubuntu and Debian ##########
	    - name: Run "apt-get update" to update the source list
	      apt: update_cache=yes
	      when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
	
	    - name: Install OpenJDK
	      apt: pkg=openjdk-7-jdk state=present
	      when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
	
	    - name: Install build-essential
	      apt: pkg=build-essential state=present
	      when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
	
	    - name: # Update all packages to the latest version
	      apt: upgrade=dist
	      when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
