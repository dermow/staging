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
