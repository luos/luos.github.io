---
layout: post
title: "Prestashop modulok fejlesztése"
date: 2011-07-01 15:00:00 +0100
comments: true
categories: prestashop, hu
---

Ezt emailben írtam, de azért gondolom valakinek hasznos lehet:

Egy modul írása alapvetően elég egyszerű.
A prestashop biztosít úgynevezett hook-okat, melyek igazából
események, és meghívódnak akkor, amikor történik valami:
http://www.daveegerton.com/prestashop-guides/Prestashop-Designers-Guide/core-modules-structure/hooks.html
Ezekre a hookokra fel lehet iratkozni, így a te kódod is lefut akkor,
amikor az esemény megtörténik.
Az új verziót annyira én se vágom, de ott már van valami trükközés a
modulinstallációnál xml fájlokkal.
Nagyjából ennyi a történet. Létrehozol egy mappát a modules mappában,
bele egy ugyanolyan nevű phpt és classt és kész a modul.
Az install metódusban feliratkozol ezekre a hookokra, majd az
adminfelületen felinstallálod és bekapcsolod(!) a modult.
Legegyszerűbb ezt kilesni egy már ottlévő modulból. A többi része a
fejlesztésnek tiszta php,  néha a prestashop Database vagy hasonló
classjait felhasználva.

Ha valami új doboz vagy ilyesmi kellett az admin felületre, akkor én
azt csak úgy tudtam megoldani, hogy belepiszkálok a viewba. Szerintem
erre nem nagyon van más megoldás.

http://www.daveegerton.com/prestashop-guides.html

Az egész nem valami bonyolult. :)
Sajna én már rég csináltam ezt, így konkrétabb linkeket nem tudok
adni, még 1.3.x-hez fejlesztettem.