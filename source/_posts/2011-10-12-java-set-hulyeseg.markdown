---
layout: post
title: "Java Set hülyeség"
date: 2011-10-12 20:17:29 +0100
comments: true
categories: java, hu
---

Csodálatos módon, a Java Set  megvalósításban, ha az ember vizsgálni szeretné, hogy egy elem benne van-e a halmazban, akkor a contains metódus, objektet vár, ami szerintem tök hülyeség, főleg azért mert nem szól, ha véletlen mást ellenőrzök.
Pl:

``` java
Set<ListItem> votma = new HashSet<ListItem>();
...
Node n = getNode(...)
if ( ! votma.contains(n) )
{...}
```

Ez simán lefordul, ami csak azért baj, mert nem vettem észre, hogy hülyeséget írtam aztán egy óráig kerestem a hibát. Másfelől viszont érthető, hisz bármilyen objektumot meg kell tudni vizsgálni, hogy benne van-e a halmazban, viszont tudható, hogy egy olyan objektum tuti nincs a halmazban, ami nem is lehet benne.




