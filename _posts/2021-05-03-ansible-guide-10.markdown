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

Bevor wir noch zu einigen Besonderheiten von Loops kommen, schauen wir uns das ganze an einem praktischen Beispiel an.
