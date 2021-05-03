---
layout: post
title:  "Ansible Starter-Guide: #010 - Loops"
date:   2021-05-03 14:30:42 +0100
categories: Ansible
---

Heyho! In diesem Teil des Ansible Tutorials beschäftigen wir uns mit Loops (Schleifen). Einen Loop setzten wir immer dann ein, wenn wir einen bestimmten Task
mehrfach wiederholen möchsten. Ein klassisches Beispiel wäre hier zum Beispiel das setzen von Dateiberechtigungen auf eine Liste von Dateien und Verzeichnissen. Bei zehn verschiedenen Pfaden bräuchten wir dafür schon 10 verschiedene Tasks, also ein ganz schöner Overhead an Code. Auch ermöglichen uns Loops die Ausführung von Tasks auf Listen, die wir uns während der Laufzeit erstellen.

Wie immer schauen wir uns das an einem Beispiel an. 



<!-- excerpt-end -->
