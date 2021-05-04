---
layout: post
title:  "Ansible Starter-Guide: #010 - Loops"
date:   2021-05-03 14:30:42 +0100
categories: Ansible
---

Heyho! In diesem Teil des Ansible Tutorials beschäftigen wir uns mit Loops (Schleifen). Einen Loop setzten wir immer dann ein, wenn wir einen bestimmten Task
mehrfach wiederholen möchsten. Ein klassisches Beispiel wäre hier zum Beispiel das setzen von Dateiberechtigungen auf eine Liste von Dateien und Verzeichnissen. Bei zehn verschiedenen Pfaden bräuchten wir dafür schon 10 verschiedene Tasks, also ein ganz schöner Overhead an Code. Mit der Hilfe von Loops können wir das in einer einzigen Task-Definition lösen. Auch ermöglichen uns Loops die Ausführung von Tasks auf Listen, die wir uns während der Laufzeit erstellen.

Schauen wir uns mal einen Task an, für den ein simpler Loop definiert ist:

```yaml
- hosts: all
  tasks:
    - name: task mit loop
      debug: 
        msg: "Ich bin: {%raw%}{{ item }}{%endraw%}"
      loop:
        - "Loop-Item 1"
        - "Loop-Item 2"
        - "Item 3"
        - "Noch ein Item"
```
<!-- excerpt-end -->

Wichtig sind hier vor Allem zwei Dinge. Zum einen der neue Parameter auf Task-Ebene "loop". Unter diesem können wir entweder (wie im Beispiel) direkt eine Liste mit Werten angeben, oder aber eine Variable nutzen. Dazu aber gleich mehr. Zum Anderen nutzen wir in der Message die Variable "{%raw%}{{ item }}{%endraw%}". Diese ist immer dann automatisch verfügbar, wenn wir einen Loop für unseren Task definieren und enthält dann je Durchlauf den entsprechenden Wert aus der Liste. 

Die Ausgabe des obigen Beispiels würde dann in etwa so aussehen:

```
PLAY [localhost] ********************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [localhost]

TASK [task mit loop] ****************************************************************************************************************************************************
ok: [localhost] => (item=Loop-Item 1) => {
    "msg": "Ich bin: Loop-Item 1"
}
ok: [localhost] => (item=Loop-Item 2) => {
    "msg": "Ich bin: Loop-Item 2"
}
ok: [localhost] => (item=Item 3) => {
    "msg": "Ich bin: Item 3"
}
ok: [localhost] => (item=Noch ein Item) => {
    "msg": "Ich bin: Noch ein Item"
}

PLAY RECAP **************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Wir haben also einen Task definiert, der pro Item unter "loop" einmal ausgefürt wird. Und zwar jedes mal mit dem entsprechenden Wert aus dem Loop in der Variable "item".

Bevor wir noch zu einigen Besonderheiten von Loops kommen, schauen wir uns das ganze an einem praktischen Beispiel an. Nehmen wir an, wir möchten auf unseren Webservern mehrere Verzeichnisse anlegen: 

* /var/www/html/my_icons
* /var/www/html/my_documents
* /var/www/html/my_other_stuff

Die 3 Werte möchten wir in diesem Beispiel nicht direkt unter dem Parameter "loop" definieren, sondern vorher in einer Variable definieren. So können wir den Inhalt z.B über die Host- und Groupvars variieren. 

Wir legen uns also zunächst eine Variable in den Groupvars für "webservers" an:

##### ~/ansible-guide/group_vars/webservers/main.yml
```yaml
---
dirs_to_create: 
  - "/var/www/html/my_icons"
  - "/var/www/html/my_documents"
  - "/var/www/html/my_other_stuff"
```

Die Variable "dirs_to_create" hat den Typ "list" und ist nun für alle Hosts in der Gruppe "webservers" verfügbar. Nun erstellen wir uns noch ein passendes Playbook:

##### ~/ansible-guide/loops1.yml
``` yaml
---
- hosts: webservers
  tasks:
    - name: create directories
      file:
        path: "{%raw%}{{ item }}{%endraw%}"
        state: directory
      loop: "{%raw%}{{ dirs_to_create }}{%endraw%}"
      become: true
      
```

Wir erstellen also einen Task mit dem Modul "file". Mit diesem können wir Dateien und Verzeichnisse verwalten. Unter "loop" geben wir in diesem Fall ncicht die Auflistung direkt, sondern die Variable "dirs_to_create" an. Ansilbe erkennt, das es sich hierbei um eine Liste handelt und wendet "loop" darauf an. Die Liste enthält alle Pfade, die wir anlgen möchten. Wir geben im Modul-Parameter "path" also die Loop-Variable "item" an und definieren mit "state: directory", dass Ansible ein Verzeichnis anlegen soll.

Dann fürhren wir das Playbook mal aus:
```bash
ansible-playbook -i inventory.txt loops1.yml
```
```
PLAY [webservers] ********************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [create directories] ***********************************************************************************************************************************************
changed: [ansible-guide-1] => (item=/var/www/html/my_icons)
changed: [ansible-guide-1] => (item=/var/www/html/my_documents)
changed: [ansible-guide-1] => (item=/var/www/html/my_other_stuff)
changed: [ansible-guide-2] => (item=/var/www/html/my_icons)
changed: [ansible-guide-2] => (item=/var/www/html/my_documents)
changed: [ansible-guide-2] => (item=/var/www/html/my_other_stuff)

PLAY RECAP **************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-guide-2                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
