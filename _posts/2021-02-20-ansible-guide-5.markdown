---
layout: post
title:  "Ansible Starter-Guide: #005 - Variablen" 
date:   2021-02-20 16:57:42 +0100
categories: Ansible
---

Heyho und willkommen zurück zur Ansible Starter-Guide. In diesem fünften Teil der Reihe möchte ich euch zeigen wie Variablen funktionieren und wie wir mit
mit ihnen mit Unterschieden zwischen verschiedenen Systemen umgehen können.

Die simpelste Anwendung von Variablen ist hierbei, mehrfach vorkommende Werte (z.B. Dateipfade) in einer Variable zu speichern. 

Lasst uns direkt mit einem Beispiel beginnen. Nehmen wir an, wir möchten eine index.html und eine style.css an unseren Webserver ausliefern. Zur besseren Übersicht 
packe ich das ganze in ein eigenes Playbook. Wer möchte kann aber einfach das webservers.yml aus dem letzten Teil erweitern.

#### Ohne Nutzung von Variablen:

```yaml
- name: Play ohne Variablen
  hosts: webservers
  tasks:
    - name: copy index.html
      copy: 
        src: files/index.hmlt
        dest: /var/www/html/index.html
      become: true
      
   - name: copy style.css
     copy:
       src: files/style.css
       dest: /var/www/html/style.css
     become: true
```

Je weiter wir unser Playbook stricken, desto häufiger werden wir den Pfad "/var/www/html" verwenden müssen. Es wäre soch super, wenn wir diesen einfach in ein Variable packen können. Dann machen wir das doch:

#### Mit Nutzung einer Variable

```yaml
- name: Play mit Variable
  hosts: webservers
  vars:
    my_docroot: /var/www/html   # <-- Definition der Variable
  tasks:
    - name: copy index.html
      copy: 
        src: files/index.hmlt
        dest: {% raw %}"{{ my_docroot }}/index.html" # <-- Nutzung der Variable {% endraw %}
      become: true
      
   - name: copy style.css
     copy:
       src: files/style.css
       dest: {% raw %}"{{ my_docroot }}/style.css" {% endraw %}
     become: true
```

Dies ist die simpelste Definitions von Variablen. Wir haben die Variable "my_docroot" ganz einfach im Playbook definiert und können auf sie nun im gesamten Play "Play mit Variable" zugreifen. Um Variablen zu verwenden nutzen wir folgendes Format "{{ variablen_name }}". Wichtig ist hier, dass wir den gesamten String dafür in Quotes (") setzen müssen.

In diesem Fall nutzen wir eine Variable des Typs "string", also eine einfache Zeichenkette. Es gibt aber noch weitere Variablen-Typen, auf die wir im Verlauf der Starter-Guide noch genauer eingehen werden. In diesem Teil werde ich zu jedem der Typen nur einen kurzen Satz verlieren, da wir im Laufe des Tutorials noch genauer darauf eingehen werden. Einige davon sind sogar ein eigenes Kapitel wert.


### Variablen-Typen

#### Strings: Zeichenketten

```yaml
my_string: Hello World!
````

#### Numbers: Ganzzahlen

Wie der Name schon sagt, ist dieser Typ für numerische Werte zuständig. 

```yaml
number_of_files: 8
```

#### Float: Gleitkommazahlen
Diesen Datentyp nutzen wir immer dann, wenn es um Gleitkommazahlen geht:

``` yaml
my_float: 2.5
```

#### List
Weiter kann man in Ansible auch einfache Listen definieren
```yaml
files_to_copy:
  - file1.txt
  - file2.txt
  - file3.txt
```

#### Dictionary:
Oder etwas komplexere "Listen"
```yaml
files_to_copy:
  - filename: file2.txt
    file_src: /opt/file2.txt
    file_dest: /home/me/file.txt
```

#### Boolean:
Schlussendlich noch der Boolean-Datentyp. Diesen nutzen wir immer dann, wenn etwas wahr oder falsch sein kann.
```yaml
copy_files: true
overwrite_files: false
```

## Host- und Groupvars

Die im ersten Beispiel definierte Variable gilt nun für alle Hosts mit dem selben Wert. Was machen wir nun aber, wenn wir Werte definieren wollen, die sich für jeden Host unterscheiden? Gehen wir zurück zu unserem Beispiel-Szenario. Nehmen wir an, der Inhalt der index.html unserer beiden Webserver soll sich pro Host unterscheiden, sodass der erste "Ich bin webserver1" und der zweite "Ich bin webserver2" ausgibt.

Dazu nutzen wir soganannte Host-Variablen (host_vars). Um diese zu nutzen, legen wir uns ein neues Verzeichnis in unserem Beispiel-Setup an:

```bash
cd ~/ansible-guide
mkdir -p host_vars/ansible-guide-1
mkdir -p host_vars/ansible-guide-2
```
In jedem Verzeichnis erstellen wir jetzt noch ein File "main.yml"

#### ~/ansible-guide/host_vars/ansible-guide-1/main.yml
```yaml
my_welcome_text: Ich bin webserver1!
```

#### ~/ansible-guide/host_vars/ansible-guide-2/main.yml
```yaml
my_welcome_text: Ich bin webserver2!
```

Wenn wir unser Playbook starten, wird ansible automatisch alle Dateien unter "host_vars" lesen und die Variablen-Werte den passenden Hosts zuordnen. Wichtig ist hierbei, dass die Namen der Verzeichnisse mit den Namen der Hosts in unserem Inventory übereinstimmen.

Um das nun zu testen, müssen wir noch eine kleine Anpassung an unserem Webserver-Playbook vornehmen. Mit dem Modul "copy" können wir nämlich anstelle einer Quell-Datei auch direkt den gewünschten Inhalt der Zieldatei definieren. Das sieht dann so aus:

#### ~/ansible-guide/webservers.yml

```yaml
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

    - name: copy index.html
      copy:
        dest: /var/www/html/index.html
        content: {%raw%}"<h1>{{ my_welcome_text }}"{%endraw%} # <-- Inhalt
      become: true
```
Beachtet hier die Änderung am letzten Task. Statt eine Quelldatei für das Copy-Modul zu definieren, geben wir direkt den gewwünschten Inhalt der
Zieldatei ein und nutzen hierfür unsere Variable.











