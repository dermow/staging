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

## Beispiel 1: Paketinstallation auf unterschiedlichen Distributionen

Nehmen wir an, wir möchten nun auf beiden Datenbankservern den PostgreSQL-Server installieren. In den vorherigen Beispielen haben wir dazu das Modul "apt" genutzt. Bei unserem Server mit dem Ubuntu wird das auch weiterhin funktionieren. CentOS nutzt zum Verwalten von Paketen allerdings ein Tool mit dem Namen "yum". Glücklicherweise bringt Ansible auch hier bereites ein Modul von Haus aus mit:

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html]

Doch wie bringen wir Ansible nun dazu, für jeden Host das richtige Modul zu nutzen. Das machen wir mit (... Trommelwirbel ...) Conditionals!
Besser gesagt, nutzen wir hier zwei Werkzeuge, die wir bereits kennen. Zum einen benötigen wir Facts, denn diese enthalten die Informationen über das Host-Betriebssystem. Zum Anderen Conditionals, umm die Tasks von den Facts abhängitg zu machen.

##### setup-postgres.yml
```yaml
- hosts: db
  gather_facts: true
  tasks:
    - name: install postgresql on debian based systems (Ubuntu)
      apt
        name: postgresql
        update_cache: true
      when: "ansible_os_family == 'Debian'"
      
    - name: install postgresql on Redhat based systems (CentOS)
      apt:
        name: postgresql
        update_cache: true
      when: "ansible_os_family == 'RedHat'"
```

Dann führen wir unser Playbook mal aus:
```bash
ansible-playbook -i inventory.txt setup-postgres.yml
```

Die Ausgabe sollte dann in etwa so aussehen:
```
PLAY [db] ********************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [ansible-guide-3]
ok: [ansible-guide-4]

TASK [install postgresql on debian based systems (Ubuntu)] **************************************************************************************************************
changed: [ansible-guide-3]
skipping: [ansible-guide-4]

TASK [install postgresql on Redhat based systems (CentOS)] **************************************************************************************************************
skipping: [ansible-guide-3]
changed: [ansible-guide-4]

PLAY RECAP **************************************************************************************************************************************************************
ansible-guide-3                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
ansible-guide-4                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
```

Wie wir sehen können, wurde für beide Install-Tasks nun jeweils der Host geskipped, bei dem die Condition nicht zutreffend war. Wir können somit also Tasks vom eingesetzten Betriebssystem abhängig machen. Besonders für Umgebungen mit verschiedenen Distributionen, macht uns dies das Leben sehr viel einfacher!

## Beispiel 2: Einen Task auf einem bestimmten Host einschränken

Ab und an kommt es vor, dass wir einen Task im Playbook nur auf einem bestimmten Host ausführen möchten.














