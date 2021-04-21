---
layout: post
title:  "Ansible Starter-Guide: #009 - Conditionals 2" 
date:   2021-04-01 16:57:42 +0100
categories: Ansible
---


Moin Moin! Wie versprochen möchte ich euch in diesem Teil nun einige Beispiele zeigen, wie wir Conditionals in Ansible verwenden können. 
Mein Test-Setup habe ich mir zu diesem Zweck um einen weiteren Host - diesmal mit einem CentOS Betriebsystem erweitert:

#### ansible-guide-4
* OS: CentOS 8
* IP: 192.168.0.14

Unser neues Inventory sieht dann so aus:

##### ~/ansible-guide/inventory.txt
```
[webservers]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12

[db]
ansible-guide-3 ansible_ssh_user=ansible ansible_host=192.168.0.13
ansible-guide-4 ansible_ssh_user=ansible ansible_host=192.168.0.14

```

Wir möchten nun also 4 Hosts mit Ansible verwalten. Drei davon auf Ubuntu-Basis und eines mit einem CentOS. Unsere beiden Webserver sind dabei identisch. Bei
den Datenbank-Servern dagegen setzen wir 2 unterschiedliche Betriebssysteme ein. 

Nehmen wir an, wir möchten nun auf beiden Datenbankservern den MySQL-Server installieren. In den vorherigen Beispielen haben wir dazu das Modul "apt" genutzt. Bei unserem Server mit dem Ubuntu wird das auch weiterhin funktionieren. CentOS nutzt zum Verwalten von Paketen allerdings ein Tool mit dem Namen "yum". Glücklicherweise bringt Ansible auch hier bereites ein Modul von Haus aus mit:

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html]








