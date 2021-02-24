---
layout: post
title:  "Ansible Starter-Guide: #005 - Variablen" 
date:   2021-02-22 16:57:42 +0100
categories: Ansible
---

Hallo zusammen. Hiermit kommen wir zu Teil 6 der Ansible Starter-Guide. In diesem Teil möchte ich mir mit euch zusammen die sogenannten Facts anschauen.

### Was sind Facts?

Vielleicht ist euch zu Beginn jedes Playbook-Durchlaufs schon folgender Part aufgefallen:

```
TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
```
Dies ist ein Task, der von Ansible standardmäßig ausgeführt wird wenn das Sammeln von Facts aktiviert ist. Facts sind Variablen, die von Ansible zum Start eines Playbooks automatisch gesetzt werden. Sie enthalten alle möglichen Informationen zum aktuellen Play, dessen Zielsystemen und der aktuellen Umgebung. 

Ein paar kleine Beispiele:

**ansible_hostname**: Enthält den von Ansible herausgefundenen Hostnamen des aktuellen Hosts

**inventory_hostname**: Enthält den im Inventory definierten Hostname des aktuellen Hosts

**ansible_default_ipv4**: Enthält die erste von Ansible gefundene, primäre IPv4-Adresse

**ansible_distribution**: Enthält den Namen der OS-Distribution des Zielhosts, z.B "Ubuntu", "Debian" oder "Suse".

Wir können uns mit einem kleinen AdHoc-Command alle verfügbaren Facts zu einem System anzeigen lassen:

```bash
ansible ansible-guide-1 -m setup
```

Die Ausgabe ist dann ein sehr großer Block im JSON (Java Script Object Notation) Format, den ich jetzt zwecks Übersichtlichkeit nicht hier einfüge. Probiert das am besten einfach selbst aus!

### Fact-Gathering aktivieren / deaktivieren
Per Default ist das Sammeln von Facts für jedes Playbook aktiviert. Allerdings kostest das auch Zeit und kann daher bei Bedarf deaktiviert werden:

```yaml
- hosts: all
  gather_facts: no   # <-- deaktiviert das Sammeln von Facts
  tasks:
    - name: task 1
      debug:
        msg: "fact gathering is off"

```

### Zugriff auf Facts in einem Playbook

Da Facts schlussendlich auch Variablen sind, ist der Zugriff auf diese identisch. So können wir uns z.B. die aktuelle Distribution und OS-Familie ausgeben lassen:

##### ~/ansible-guide/playbook-with-facts.yml
```yaml
- hosts: all
  tasks:
    - name: print current distribution
      debug:
        msg: "My distribution is {{ ansible_distribution }}"

    - name: print current os family
      debug:
        msg: "My os family is {{ ansible_os_family }}"

```

#### Zusammenfassung
Damit kommen wir schon zum Ende dieses kurzen Kapitels in dem wir gelernt haben, was Facts sind und wie wir auf diese zugreifen können. Wir werden Facts im Laufe dieses Guides noch sehr häufig in praktischen Beispieln nutzen, unter Anderem im nächtsten Teil, in dem wir uns Conditionals (Bedingungen) anschauen werden.

Bis bald!
Der Mow

