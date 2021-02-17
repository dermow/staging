---
layout: post
title:  "Ansible Starter-Guide: #004 - Playbooks, Tasks und Handler"
date:   2021-02-14 16:01:42 +0100
categories: Ansible
---

Wilkommen zurück, in diesem Teil werden wir einige wichtige Bestandteile von Ansible näher kennenlernen. 

## Playbooks

Mit Playbooks können wir Tasks wiederverwendbar definieren. Ein Playbook besteht dabei aus einem oder mehreren Plays. Wir könnten z.B ein Playbook mit dem Namen "webserver.yml" nutzen um unseren Webserver damit hochzuziehen.

Lasst uns versuchen, die Struktur eines Playbooks anhand des folgenden Beispiels zu lernen:

```yaml
1 - name: webserver install 
2   hosts: webservers
3   tasks: 
4     - name: install apache2
5       apt:
6         name: apache2
7         state: present
8         update_cache: yes
9       become: true
10
11    - name: enable and start apache2 systemd service
12      systemd:
13        name: apache2
14        enabled: true
15        state: started

```

Gehen wir das ganze Zeile für Zeile durch. 

## Zeile 1 - Name
Im Feld "name" in Zeile 1 kann ein beschreibender Name für den Play vergeben werden. Dieser Parameter ist optional, ich empfehle aber jeden Play zu benennen.

## Zeile 2 - Zieldefinition
In Zeile 2 definieren wir mit dem Feld "hosts" die Zielsyteme für diesen Play. Hier haben wir mehrere Möglichkeiten:

#### Gruppe(n)

Wir können eine oder mehrere Gruppen aus unserem Inventory als Ziel definieren:

```yaml
- name: Play der auf eine Gruppe ausgeführt wird
  hosts: Gruppe1
  tasks:
    - name: Beispieltask1
      apt:
        name: apache2
        state: present
```

```yaml
- name: Play der auf mehrere Gruppen ausgeführt wird
  hosts: 
    - Gruppe1
    - Gruppe2
  tasks:
    - name: Beispieltask1
      apt:
        name: apache2
        state: present
```

#### Hosts

Alternativ können wir auch direkt die Hosts aus dem Inventory als Ziel angeben:

```yaml
- name: Play der auf einem Host ausgeführt wird
  hosts: webserver1
  tasks:
    - name: Beispieltask1
      apt:
        name: apache2
        state: present
```

```yaml
- name: Play der auf mehreren Hosts ausgeführt wird
  hosts: 
    - webserver1
    - webserver2
  tasks:
    - name: Beispieltask1
      apt:
        name: apache2
        state: present
```

Es gibt noch viele weitere Möglichkeiten, mit denen wir in diesem Feld die Ziel-Liste beeinflussen können. Wir können z.B einzelne Host oder Gruppen wieder von der Zieldefinition ausnehmen (Stichwort 'Patterns') Dazu aber in einem späteren Artikeln mehr.


## Zeile 3 - Tasks

Hier beginnt die Liste der einzelnen Task für diesen Play.

## Ab Zeile 4 - Task-Struktur

Hiermit kommen wir zur Struktur eines Tasks. Achtet hier auf die Einrückung der einzelnen Elemente. Wir unterscheiden hier zwischen Parametern die zum Task gehören, und Parametern die zum Modul gehören.

Das Feld "name" (Zeile 4) ist wie auch beim Play optional aber dringend empfohlen, da es auch bei der Ausgabe erscheint. Schließlich wollen wir ja gerne wissen, was gerade passiert. In Zeile 5 steht der Name des Moduls, das wir nutzen wollen. In diesem Fall wollen wir das Modul "apt" nutzen um ein Paket zu installieren. Alles was jetzt eine Einrückung weiter steht ist ein Parameter auf Modul-Ebene, gilt also für das Modul und nicht für den Task selbst. Das ist ein Punkt, bei dem sehr viele zunächst ihre Probleme haben. Mit der Zeit prägt man sich das aber ganz gut ein! 

Zur einfacheren Übersicht, paste ich unser Beispiel-Playbook hier noch einmal:

```yaml
1 - name: webserver install 
2   hosts: webservers
3   tasks: 
4     - name: install apache2
5       apt:
6         name: apache2
7         state: present
8         update_cache: yes
9       become: true
10
11    - name: enable and start apache2 systemd service
12      systemd:
13        name: apache2
14        enabled: true
15        state: started
16      become: true

```

Wir haben also einen Task mit dem Namen "install apache2". Dieser Task nutzt das Modul apt (Zeile 5). Für das apt-Modul definieren wir widerum 3 Parameter (Zeile 6-8). 
* name = Name des zu installierenden Pakets
* state = Ziel-Zustand. present = installiert, absent = deinstalliert, latest = aktuellste Version
* update_cache = erzwingt das vorherige Erneuern des Paket-Zwischenspeichers

In Zeile 9 folgt hier nochmal die Definition eines Parameters auf Task-Ebene (Achtet auf die Einrückung). Mit "become" geben wir im Prinzip an, dass für die Ausführung dieses Tasks Superuser-Rechte benötigt werden. Diesen Parameter werden wir sehr häufig brauchen, ich werde dem Thema daher einen eigenen Teil in dieser Reihe widmen.

Die selbe Struktur hat der 2. Task ab Zeile 11. Wir deklarieren erneut einen Task und verbeben hier den Namen "enable and start apache2 systemd service", rufen hier das Modul "systemd" auf, mit dem wir (Trommelwirbel) systemd Dienste verwalten können.
Für das Modul legen wir 3 Parameter fest (Zeile 13-15).

Zu Modul-Parametern ist wichtig zu wissen, dass es einige gibt die optional sind und einige die zwingend angegeben werden müssen. Hier hilft uns aber die Ansible-Dokumentation sehr zuverlässig weiter. Für unser Beispiel finden wir die Informationen auf diesen Seiten:

[Doku zum Modul apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
[Doku zum Modul systemd](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html)

## Beispiel

Lasst uns das Gelernte nun auf unser Beispiel-Szenario übertragen. Nehmen wir an unsere ersten beiden Hosts sollen Webserver werden, der dritte wird Datenbank-Host:

```ini
[webservers]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12

[db]
ansible-guide-3 ansible_ssh_user=ansible ansible_host=192.168.0.13
```

Aufteilen möchte ich das ganze nun in zwei Playbooks. Je nach eigenen Preferenzen und Anwendungsfall, könnten wir auch zwei Plays innerhalb eines Playbooks definieren.

### webservers.yml
``` yaml
- name: webserver setup
  hosts: webservers
  tasks:
    - name: Apache2 Setup
      apt:
        name: apache2
        state: present
        update_cache: true
      become: true

    - name: start and enable apache2
      systemd:
        name: apache2
        state: started
        enabled: true
      become: true
```

Folgendermaßen können wir das Playbook nun ausführen:
```bash
cd ~/ansible-guide
ansible-playbook -i inventory.txt playbook.yml
```

```
PLAY [webserver setup] *********************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [Apache2 Setup] ***********************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

TASK [start and enable apache2] ************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]


PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
ansible-guide-2                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

```
Interessant ist hier, dass der Task "start and enable apache2" den Status "ok" liefert und nicht wie erwartet "changed". Dies kommt daher, dass das Aktivieren und starten des Services bereits im Installationspaket definiert ist. Für Ansible gibt es in dem Moment also kein Todo.

Nun können wir noch kurz testen, ob unsere Installation auch funktioniert hat, indem wir eine der IP-Adressen in den Browser eingeben. Wir müssten jetzt die Standard-Seite des Apache Webservers zu sehen bekommen.


### database.yml
``` yaml
- name: database setup
  hosts: db
  tasks:
    - name: MySQL Setup
      apt:
        name: mysql-server
        state: present
        update_cache: true
      become: true

    - name: start and enable MySQL
      systemd:
        name: mysql
        state: started
        enabled: true
      become: true
```

Gleiches machen wir nun noch mit dem Playbook für den Datenbankserver:

```bash
cd ~/ansible-guide
ansible-playbook -i inventory.txt database.yml
```
```
PLAY [database setup] **********************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-3]

TASK [MySQL Setup] *************************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-3]

TASK [start and enable MySQL] **************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-3]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-3                 : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
