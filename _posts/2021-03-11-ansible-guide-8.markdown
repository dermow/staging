---
layout: post
title:  "Ansible Starter-Guide: #008 - Conditionals" 
date:   2021-03-11 16:57:42 +0100
categories: Ansible
---

Halli Hallo! Nach längerer Pause geht es nun mit Teil 8 des Ansible Starter-Guides weiter. Ich möchte euch in diesem Post zeigen, was Conditionals sind
und wie wir diese in Playbooks nutzen können.

### Was sind Conditionals?

Conditionals sind Bedingungen, an die wir die Ausführung von Tasks knüpfen können. Zum Beispiel können wir die einen Task nur dann ausführen lassen, wenn ein der
Ziel-Host ein bestimmtes Betriebssystem installiert hat. Wenn wir Conditionals nutzen, greifen wir bei deren Definition auf Variablen und Facts zurück.

### Ein erstes Beispiel

Lasst uns mit einem sehr einfachen Beispiel beginnen. 

##### simple-conditional.yml

```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 6
  tasks:
    - name: task mit condition
      debug:
        msg: "Variable ist größer als 5!!"
      when: "my_number > 5"
```

Conditionals werden im Parameter "when" definiert. Dieser ist ein Parameter auf Task-Ebene. Unter "when" kann dann eine, oder auch eine Liste mit Conditionals stehen. Die Conditions werden vor der Ausführung des Tasks geprüft. Sollten sie zutreffen, wird der Task ausgeführt, falls nicht wird er übersprungen und erhält den Status "skipped".

Was heißt das für das obige Beispiel-Playbook? Wir haben oben eine Variable "my_number" definiert, die den Wert 6 hat. Dann haben wir einen Task definiert, der die folgende Condition hat:

```yaml
...
when: "my_number > 5"
---
```

Wir wollen den Task also nur dann ausführen, wenn die Variable "my_number" größer ist als 5. In diesem Fall sollte der Task also ausgeführt werden. Dann lasst uns das Playbook doch einmal ausführen:

```bash
ansible-playbook -i inventory.txt simple-conditional.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Variable ist größer als 5!!"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```



