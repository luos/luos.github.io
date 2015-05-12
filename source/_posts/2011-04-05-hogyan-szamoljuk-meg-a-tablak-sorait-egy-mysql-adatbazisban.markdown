---
layout: post
title: "Hogyan számoljuk meg a táblák sorait egy MySQL adatbázisban?"
date: 2011-04-05 12:38:00 +0100
comments: true
categories: sql
---

Nem tudom, hogy ez a legjobb mód rá, mindenesetre, ha gyorsan kell, működik:

Először is, csináljunk egy lekérdezést:

``` sql

SELECT CONCAT( ' ( SELECT "', table_name, '" as tabla , COUNT(*) as db FROM ', table_name, ') UNION ALL' )
FROM information_schema.tables
WHERE table_schema = 'ADATBAZIS_NEVE'
LIMIT 0 , 300000

```

Majd ha lefuttattuk, kapunk valami ilyesmit:

``` sql

( SELECT "kutyuk_access" as tabla , COUNT(*) as db FROM kutyuk_access) UNION ALL( SELECT "kutyuk_accessory" as tabla , COUNT(*) as db FROM kutyuk_accessory) UNION ALL( SELECT "kutyuk_address" as tabla , COUNT(*) as db FROM kutyuk_address) UNION ALL( SELECT "kutyuk_alias" as tabla , COUNT(*) as db FROM kutyuk_alias) UNION ALL

```

A legutolsó SQL végéről töröljük le az UNION ALL-t, és hogy később tudjuk rendezni mondjuk