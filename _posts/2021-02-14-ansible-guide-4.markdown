---
layout: post
title:  "Ansible Starter-Guide: #004 - Playbooks, Tasks und Handler"
date:   2021-02-17 10:01:42 +0100
categories: Ansible
---

Wilkommen zurück zur Ansible Starter-Guide. In diesem Teil des Tutorials will ich versuchen, euch den Umgang mit Tasks und Playbooks näher zu bringen. Auch möchte ich euch zeigen, was Handler sind und wie wir diese einsetzen.

## Playbooks

Mit Playbooks können wir Tasks wiederverwendbar definieren. Ein Playbook besteht dabei aus einem oder mehreren Plays. Wir könnten z.B ein Playbook mit dem Namen "webserver.yml" nutzen um damit unsere Webserver zu verwalten.

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

## Taskname (Zeile 1)
Im Feld "name" in Zeile 1 kann ein beschreibender Name für den Play vergeben werden. Dieser Parameter ist optional, ich empfehle aber jeden Play möglichst sinnvoll zu benennen.

## Zieldefinition (Zeile 2)
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


## Tasks (Zeile 3)

Hier beginnt die Liste der einzelnen Task für diesen Play.

## Task-Struktur (Zeile 4)

Hiermit kommen wir zur Struktur eines Tasks. Achtet hier auf die Einrückung der einzelnen Elemente. Wir unterscheiden zwischen Parametern die zum Task gehören, und Parametern die zum Modul gehören.

Das Feld "name" (Zeile 4) ist wie auch beim Play optional aber dringend empfohlen, da es auch bei der Ausgabe erscheint. Schließlich wollen wir ja gerne wissen, was gerade passiert. In Zeile 5 steht der Name des Moduls, das wir nutzen wollen. In diesem Fall wollen wir das Modul "apt" nutzen um ein Paket zu installieren. Alles was jetzt eine Einrückung weiter steht ist ein Parameter auf Modul-Ebene, gilt also für das Modul und nicht für den Task selbst. Das ist ein Punkt, bei dem sehr viele zunächst ihre Probleme haben. 

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
* **name**: Name des zu installierenden Pakets
* **state**: Ziel-Zustand. present = installiert, absent = deinstalliert, latest = aktuellste Version
* **update_cache**: erzwingt das vorherige Erneuern des Paket-Zwischenspeichers

In Zeile 9 folgt hier nochmal die Definition eines Parameters auf Task-Ebene (Achtet auf die Einrückung). Mit "become: true" geben wir im Prinzip an, dass für die Ausführung dieses Tasks Superuser-Rechte benötigt werden. Diesen Parameter werden wir sehr häufig brauchen, ich werde dem Thema daher einen eigenen Teil in dieser Reihe widmen.

Die selbe Struktur hat der 2. Task ab Zeile 11. Wir deklarieren erneut einen Task und verbeben hier den Namen "enable and start apache2 systemd service", rufen hier das Modul "systemd" auf, mit dem wir (...Trommelwirbel...) systemd Dienste verwalten können.
Für das Modul legen wir 3 Parameter fest (Zeile 13-15).

*  **name**:    Name des zu verwaltenden Services
*  **enabled**: Legt fest, ob der Service aktiviert werden soll. Aktivierte Services werden beim System-Boot automatisch gestartet.
*  **state**:   Gibt den gewünschten Zielzustand des Services an. In umserem Fall also "started".


### Hinweis zu Modul-Parametern
Zu Modul-Parametern ist wichtig zu wissen, dass es einige gibt die optional sind und einige die zwingend angegeben werden müssen. Hier hilft uns aber die Ansible-Dokumentation sehr zuverlässig weiter. Für unser Beispiel finden wir die Informationen auf diesen Seiten:

[Doku zum Modul apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
[Doku zum Modul systemd](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html)

## Beispiel

Lasst uns das Gelernte nun auf unser Beispiel-Szenario übertragen. Nehmen wir an unsere ersten beiden Hosts sollen Webserver werden, der dritte wird Datenbank-Host. 

Wir passen unsere inventory.txt also entsprechend an:

#### ~/ansible-guide/inventory.txt

```ini
[webservers]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12

[db]
ansible-guide-3 ansible_ssh_user=ansible ansible_host=192.168.0.13
```

Aufteilen möchte ich das ganze nun in zwei Playbooks. Je nach eigenen Preferenzen und Anwendungsfall, könnten wir auch zwei Plays innerhalb eines Playbooks definieren.

#### webservers.yml
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
ansible-playbook -i inventory.txt webservers.yml
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
Interessant ist hier, dass der Task "start and enable apache2" den Status "ok" liefert und nicht wie erwartet "changed". Dies kommt daher, dass das Aktivieren und Starten des Services bereits im Installationspaket definiert ist. Für Ansible gibt es in dem Moment also kein Todo.

Nun können wir noch kurz testen, ob unsere Installation auch funktioniert hat, indem wir eine der IP-Adressen in den Browser eingeben. Wir müssten jetzt die Standard-Seite des Apache Webservers zu sehen bekommen.


#### database.yml
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

## Eigene index.html ausrollen

Wir haben jetzt per Ansible eine Standard-Installation des Apache2 und MySQL durchgeführt. Im nächsten Schritt möchte ich euch zeigen, wie wir z.B eine eigene index.html Datei ausrollen können.

Dazu legen wir und die Datei zunächst lokal auf dem Ansible-Controller an:

``` bash
cd ~/ansible-guide
mkdir files
echo "<h1>Meine eigene index.html!</h1>" > files/index.html 
```

Die Datei sieht dann so aus:

```html
<h1>Meine eigene index.html!</h1>
```

Nun erweitern wir unser Playbook "webservers.yml" um einen Task:

#### webservers.yml
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

    - name: copy index.html
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
      become: true
```

Wir machen uns hier das Ansible-Modul "copy" zu nutze. Mit diesem können Dateien vom Ansible-Controller auf die Zielsysteme kopiert werden. Dem Modul geben wir 2 Parameter mit:

* **src**: Pfad zur Quelldatei auf dem Ansible-Controller
* **dest**: Zielpfad auf den Remote-Hosts

Auch teilen wir Ansible mit "become: true" wieder mit, dass für diesen Vorgang Rootrechte benötigt werden.

Wir führen das aktualisierte Playbook nun also aus:

```bash
ansible-playbook -i inventory.txt webservers.yml
```

```
PLAY [webserver setup] *******************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [Apache2 Setup] *********************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [start and enable apache2] **********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [copy index.html] *******************************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ansible-guide-2                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

Das Resultat können wir mit curl sehen:

```bash
apt install -y curl
curl http://192.168.0.11
```
```
<h1>Meine eigene index.html!</h1>
```

Alternativ könnt ihr die IP natürlich auch über den Browser eurer Wahl aufrufen!

## Handler

Wir haben jetzt also unseren Webserver und eine Datenbank installiert. Weiter haben wir eigenen Inhalt auf unsere Webserver ausgeliefert. Zum Abschluss möchte ich euch noch zeigen, wie wir auch die Apache Konfigurationsdatei über Ansible verwalten können. Dabei wollen wir den Webserver bei Änderungen an dieser (und nur dann!) neu starten. Dafür gibt es Handler.

Wir legen uns die Apache-Konfiguration also wieder auf dem Ansible-Controller ab. Dafür hab ich mir von einem der Test-Hosts einfach den Inhalt von /etc/apache2/apache2.conf kopiert und die Kommentarzeilen etwas reduziert um das Ganze etwas übersichtlicher zu machen.

Die Datei legen wir wieder in unserem Unterordner "files" ab:

##### files/apache2.conf

```
DefaultRuntimeDir ${APACHE_RUN_DIR}
PidFile ${APACHE_PID_FILE}

Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}

HostnameLookups Off

ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn

IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

Include ports.conf

<Directory />
	Options FollowSymLinks
	AllowOverride None
	Require all denied
</Directory>

<Directory /usr/share>
	AllowOverride None
	Require all granted
</Directory>

<Directory /var/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>

AccessFileName .htaccess

<FilesMatch "^\.ht">
	Require all denied
</FilesMatch>

LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf
```

Wie erweitern unser Playbook um einen weiteren Task:

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

    - name: copy index.html
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
      become: true

    - name: copy apache2.conf
      copy:
        src: files/apache2.conf
        dest: /etc/apache2/apache2.conf
      become: true
```

Würden wir das ganze jetzt erneut ausführen, würde Ansible die apache2.conf wie auch schon die index.html auf die Zielsysteme kopieren. Wir wollen ja aber auch den automatischen Restart des Apache-Service bei Änderungen implementieren. Deshalb fügen wir nun noch einen Handler hinzu und rufen diesen im neuen Task auch direkt auf. Die wichtigsten Zeilen hab ich mit Kommentaren versehen:

### webservers.yml
``` yaml
- name: webserver setup
  hosts: webservers
  handlers:                                  # <--- Liste mit Handlern
    - name: restart-apache                   # <--- Handler-Definition
      systemd:
        name: apache2
        state: restarted
      become: trure
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
        src: files/index.html
        dest: /var/www/html/index.html
      become: true

    - name: copy apache2.conf
      copy:
        src: files/apache2.conf
        dest: /etc/apache2/apache2.conf
      notify: restart-apache                 #<-- Benachrichtigung des Handlers
      become: true
```

Wir haben nun eine weitere Liste, auf der gleichen Ebene wie "tasks", angelegt. Diese beinhaltet unsere Handler-Definitionen. Ein Handler hat exakt den selben Aufbau wie ein Task, wird beim Aufruf des Playbooks aber nicht automatisch ausgeführt. Das passiert nur wenn mindestens zwei Dinge gegeben sind:

1) Ein Task hat den Namen unseres Handlers - hier "restart-apache" - im Parameter "notify" definiert.
2) Mindestens ein Task, der Punkt 1 erfüllt liefert den Status "changed" zurück. Nur dann werden Handler ausgeführt.

Wir führen das Playbook nun also erneut aus:

```bash
ansible-playbook -i inventory.txt webserver.yml
```

```
PLAY [webserver setup] *******************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [Apache2 Setup] *********************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [start and enable apache2] **********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [copy index.html] *******************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [copy apache2.conf] *****************************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

RUNNING HANDLER [restart-apache] *********************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
ansible-guide-2                  : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Die apache2.conf wird nun also ebenfall per Ansible verwaltet und der Dienst bei Bedarf neu gestartet! Probiert das gerne noch ein paar Mal aus, indem ihr einige Parameter in der Konfiguration ändert und das Playbook ausführt.

## Wichtiger Hinweis zu Handlern
Wie in der Ausgabe oben zu sehen, werden Handler immer am Ende des Plays ausgeführt und nicht direkt nach dem aufrufenden Task. Das stellt sicher, dass z.B der Apache Restart nur einmal ausgeführt wird, auch wenn mehrere Tasks Änderungen vornehmen und den Handler aufrufen.

Möchte man explizit einen Restart an dieser Stelle haben, muss man den Restart als Task definieren.

## Das wars schon wieder!

Das war bisher der längste Teil des Tutorials und ihr braucht sicher Zeit, das Gelernte erstmal zu verarbeiten und am besten einfach selbst mit weiteren Beispielen auszuprobieren. Wie immer freue ich mich über Feedback und Verbesserungsvorschläge. Gerne hier in der Kommentarfunktion, aber auch per E-Mail an: <a href='mailto:blog.mow@gmail.com'>blog.mow@gmail.com

Im nächsten Teil möchte ich euch ein paar weitere, sehr nützliche Module vorstellen.

Ich wünsche euch viel Spaß beim selbst ausprobieren!

Der Mow


