---
title: Fuzzing JavaScript and WebAssembly engine
date: 2023-10-09
categories: [v8, fuzzing]
tags: [Fuzzing, v8, Linux] #minúsculas
---

Bé, de moment començarem amb català a aquest post. Primer de tot, dir que havia deixat el blog abandonat, la veritat és que he estat fent l'OSCP i l'OSWP.

# Què és el Fuzzing?

La gent, majoritàriament, coneix el fuzzing web que bàsicament consisteix a provar rutes mitjançant un diccionari.


Sovint els resultats del fuzzing de binaris revelen falles de seguretat com ara el desbordament de memòria (buffer overflow), desbordament de la pila (stack overflow), o desbordament de l'heap (heap overflow).
