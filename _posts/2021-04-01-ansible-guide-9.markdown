---
layout: post
title:  "Ansible Starter-Guide: #009 - Conditionals 2" 
date:   2021-04-01 16:57:42 +0100
categories: Ansible
---


Moin Moin! Wie versprochen möchte ich euch in diesem Teil nun einige Beispiele zeigen, wie wir Conditionals in Ansible verwenden können. 
Mein Test-Setup habe ich mir zu diesem Zweck um einen weiteren Host - diesmal mit einem Open Suse Betriebsystem erweitert:

#### ansible-guide-4
* OS: openSUSE Leap 15.1
* IP: 192.168.0.14

Unser neues Inventory sieht dann so aus:

```
[webservers]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12

[db]
ansible-guide-3 ansible_ssh_user=ansible ansible_host=192.168.0.13
ansible-guide-4 ansible_ssh_user=ansible ansible_host=192.168.0.14

```




