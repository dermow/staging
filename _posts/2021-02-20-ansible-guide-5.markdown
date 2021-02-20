---
layout: post
title:  "Ansible Starter-Guide: #005 - Variablen und Facts"
date:   2021-02-20 16:57:42 +0100
categories: Ansible
---

Heyho und willkommen zurück zur Ansible Starter-Guide. In diesem fünften Teil der Reihe möchte ich euch zeigen wie Variablen funktionieren und wie wir mit
mit ihnen mit Unterschieden zwischen verschiedenen Systemen umgehen können. Auch möchte ich euch den Nutzen von sogenannten Facts erklären. 

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
        dest: "{%raw}{{ my_docroot }}{%endraw}/index.html" # <-- Nutzung der Variable
      become: true
      
   - name: copy style.css
     copy:
       src: files/style.css
       dest: "{{ my_docroot }}/style.css"
     become: true
```

Dies ist die simpelste Definitions von Variablen. Wir haben die Variable "my_docroot" ganz einfach im Playbook definiert und können auf sie nun im gesamten Play "Play mit Variable" zugreifen. Um Variablen zu verwenden nutzen wir folgendes Format "{{ variablen_name }}". Wichtig ist hier, dass wir den gesamten String dafür in Quotes (") setzen müssen.

In diesem Fall nutzen wir eine Variable des Typs "string", also eine einfache Zeichenkette. Es gibt aber noch weitere Variablen-Typen, auf die wir im Verlauf der Starter-Guide noch genauer eingehen werden. Aktuell kennt Ansible die folgenden Typen:


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

Auf jeden der einzelnen Datentypen werden wir im Laufe des Tutorials noch genauer eingehen. Einigen werde ich wahrscheinlich sogar ein eigenes Kapitel widmen. In diesem Teil möchte ich aber zunächst die allgemeine Funktion von Variablen behandeln.













