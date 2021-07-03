---
layout: post
title:  "Ansible Starter-Guide: #012 - Loops 2 & Dictionaries"
date:   2021-07-03 14:30:42 +0100
categories: Ansible
---

Heyho! Nach etwas längerer Pause bin ich nun endlich mal wieder zum bloggen gekommen. Wie versprochen also hier der nächste Teil des Ansible Starter-Guide,
der euch weitere Informationen zur Verwendung von Loops näher bringen wird.

Bevor wir aber zu unseren Loops zurückkehren möchte ich euch noch kurz Dictionaries vorstellen, die bisher nur in Teil 5 kurz erwähnt wurden. Dictionaries sind
einfach ausgedrückt Listen, dren Items mehrere Felder besitzen können. Ein einfaches Beispiel:

```yaml
people:
  - name: Max
    nachname: Mustermann
    alter: 21
  - name: Julia
    nachname: Roberts
    alter: 40
  - name: Albus
    nachname: Dumbledore
    alter: 99
```

Wie in einer simplen "list" markiert der Bindestrich den Beginn eines neuen Items. In diesem Beispiel besitzt jedes der beiden Items die Felder "name", "nachname" und "alter".

Auch dictionaries können wir in Loops verwenden. Möchten wir zum Beispiel die in userem Beispiel definierten Informationen ausgeben, sähe das Playbook so aus:

##### print_dictionary.yml 
``` yaml
{%raw%}
- hosts: localhost
  vars:
    people:
      - name: Max
        nachname: Mustermann
        alter: 21
      - name: Julia
        nachname: Roberts
        alter: 40
      - name: Albus
        nachname: Dumbledore
        alter: 99
       
  tasks:
    - name: list people
      debug:
        msg: "Vorname: {{ item.name }}, Nachname: {{ item.nachname }}, Alter: {{ item.alter }}"
      loop: "{{ people }}"
{%endraw%}
```


